<template>
  <div id="body">
    <div class="col">
      <div ref="first" style="background: red" @click="startMap('first')"></div>

      <div ref="second" style="background: green" @click="startMap('second')">
        <div class="absolute-square" @click.prevent="updateMap('second')">
          <span v-if="maps.second">
            {{ maps.second.currentMarkerId }}
          </span>
        </div>
      </div>

      <div ref="third" style="background: blue" @click="startMap('third')">
        <div class="absolute-square" @click.prevent="updateMap('third')">
          <span v-if="maps.third">
            {{ maps.third.currentMarkerId }}
          </span>
        </div>
        <div
          class="absolute-square"
          style="left: auto; right: 50px"
          @click.prevent="moveCamera('third')"
        >
          moveCamera
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { CapacitorGoogleMaps } from '@capacitor-community/capacitor-googlemaps-native';

import style from '../../common/style.json';

export default {
  name: 'App',
  components: {},
  data() {
    return {
      maps: {},
    };
  },
  computed: {
    style() {
      return JSON.stringify(style);
    },
  },
  methods: {
    getRandomArbitrary(min, max) {
      return Math.random() * (max - min) + min;
    },
    async moveCamera(refName) {
      const googleMap = this.maps[refName];

      if (!googleMap) {
        return;
      }

      const result = await CapacitorGoogleMaps.moveCamera({
        mapId: googleMap.mapId,
        cameraPosition: {
          target: {
            latitude: -33.86,
            longitude: 151.2,
          },
          zoom: 0,
        },
        duration: 1000,
      });

      alert(JSON.stringify(result));
    },
    async updateMap(refName) {
      const googleMap = this.maps[refName];

      if (!googleMap) {
        return;
      }

      const dotShown = confirm('Show dot?');

      const result = await CapacitorGoogleMaps.updateMap({
        mapId: googleMap.mapId,
        preferences: {
          gestures: {
            isScrollAllowedDuringRotateOrZoom: true,
          },
          appearance: {
            type: refName === 'second' ? 4 : null,
            isMyLocationDotShown: dotShown,
          },
        },
      });

      alert(JSON.stringify(result));
    },
    async startMap(refName) {
      // if map already (being) initialized, avoid a new map being initialized
      if (this.maps[refName]) {
        return;
      }

      // find element where element should be attached to
      const element = this.$refs[refName];
      if (!element) {
        return;
      }

      // prevent race condition, where firing `startMap` quickly after one another, starts up multiple maps
      this.maps[refName] = {};

      // initialize map
      await CapacitorGoogleMaps.initialize({
        key: null,
        devicePixelRatio: window.devicePixelRatio,
      });

      // remove background, so map can be seen
      element.style.background = '';

      // get boundaries of element
      const boundingRect = element.getBoundingClientRect();

      // finally create the map
      try {
        const result = await CapacitorGoogleMaps.createMap({
          boundingRect: {
            width: Math.round(boundingRect.width),
            height: Math.round(boundingRect.height),
            x: Math.round(boundingRect.x),
            y: Math.round(boundingRect.y),
          },
          cameraPosition: {
            target: {
              latitude: -33.86,
              longitude: 151.2,
            },
            zoom: 12,
          },
          preferences: {
            gestures: {
              isScrollAllowedDuringRotateOrZoom: false,
            },
            appearance: {
              style: refName === 'first' ? this.style : null,
              isMyLocationDotShown: true,
            },
          },
        });

        alert(JSON.stringify(result));

        // remember mapId for curent element for later use
        this.maps[refName] = {
          mapId: result?.googleMap?.mapId,
        };

        // set `data-maps-id` attribute for delegating touch events
        element.setAttribute('data-maps-id', result.googleMap.mapId);

        alert('Map loaded successfully');

        if (result?.googleMap?.mapId) {
          var i;
          for (i = 0; i < 500; i++) {
            CapacitorGoogleMaps.addMarker({
              mapId: result.googleMap.mapId,
              latitude: -this.getRandomArbitrary(33, 34),
              longitude: this.getRandomArbitrary(151, 152),
              title: 'Some title',
              snippet: 'Some snippet ',
              metadata: {
                some: 'data',
              },
            });
          }

          CapacitorGoogleMaps.didTapInfoWindow(
            {
              mapId: result.googleMap.mapId,
            },
            (result) => {
              alert('didTapInfoWindow: ' + JSON.stringify(result));
            }
          );

          CapacitorGoogleMaps.didCloseInfoWindow(
            {
              mapId: result.googleMap.mapId,
            },
            (result) => {
              // this will never fire, because it is overwritten by the next `didCloseInfoWindow`
              alert('this will never be shown');
            }
          );

          CapacitorGoogleMaps.didCloseInfoWindow(
            {
              mapId: result.googleMap.mapId,
            },
            (result) => {
              // but this will fire
              alert('didCloseInfoWindow: ' + JSON.stringify(result));
              this.maps[refName].currentMarkerId = null;
            }
          );

          CapacitorGoogleMaps.didTapMap(
            {
              mapId: result.googleMap.mapId,
            },
            (result) => {
              alert('didTapMap: ' + JSON.stringify(result));
            }
          );

          CapacitorGoogleMaps.didLongPressMap(
            {
              mapId: result.googleMap.mapId,
            },
            (result) => {
              alert('didLongPressMap: ' + JSON.stringify(result));
            }
          );

          CapacitorGoogleMaps.didTapMarker(
            {
              mapId: result.googleMap.mapId,
              preventDefault: false, // set to true if you do not want the default behaviour
            },
            (result) => {
              this.maps[refName].currentMarkerId = result.marker.markerId;
              alert('didTapMarker: ' + JSON.stringify(result));
            }
          );

          CapacitorGoogleMaps.didTapMyLocationButton(
            {
              mapId: result.googleMap.mapId,
              preventDefault: false, // set to true if you do not want the default behaviour
            },
            (result) => {
              alert('didTapMyLocationButton: ' + JSON.stringify(result));
            }
          );

          CapacitorGoogleMaps.didTapMyLocationDot(
            {
              mapId: result.googleMap.mapId,
            },
            (result) => {
              alert('didTapMyLocationDot: ' + JSON.stringify(result));
            }
          );
        }
      } catch (e) {
        alert('Map failed to load');
        this.maps[refName] = null;
      }
    },
  },
};
</script>

<style>
body {
  margin: 0;
}

div {
  position: relative;
  height: 100%;
  width: 100%;
}

#body {
  height: 100vh;
  width: 100vw;
  color: #000;
}

.col {
  display: flex;
  flex-direction: column;
}

.absolute-square {
  position: absolute;
  top: 0;
  left: 0;
  width: 100px;
  height: 100px;
  background: purple;
}
</style>
