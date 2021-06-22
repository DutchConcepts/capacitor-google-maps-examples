<template>
  <div id="body">
    <div class="col">
      <div ref="first" style="background: red" @click="startMap('first')">
        <div class="absolute-square">
          <span v-if="maps.first">
            {{ maps.first.currentMarkerId }}
          </span>
        </div>
      </div>
      <div ref="second" style="background: green" @click="startMap('second')">
        <div class="absolute-square">
          <span v-if="maps.second">
            {{ maps.second.currentMarkerId }}
          </span>
        </div>
      </div>
      <div ref="third" style="background: blue" @click="startMap('third')">
        <div class="absolute-square">
          <span v-if="maps.third">
            {{ maps.third.currentMarkerId }}
          </span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { CapacitorGoogleMaps } from '@capacitor-community/capacitor-googlemaps-native';

export default {
  name: 'App',
  components: {},
  data() {
    return {
      maps: {},
    };
  },
  methods: {
    getRandomArbitrary(min, max) {
      return Math.random() * (max - min) + min;
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
        const map = await CapacitorGoogleMaps.create({
          width: Math.round(boundingRect.width),
          height: Math.round(boundingRect.height),
          x: Math.round(boundingRect.x),
          y: Math.round(boundingRect.y),
          latitude: -33.86,
          longitude: 151.2,
          zoom: 12,
        });

        // remember mapId for curent element for later use
        this.maps[refName] = {
          mapId: map?.mapId,
        };

        // set `data-maps-id` attribute for delegating touch events
        element.setAttribute('data-maps-id', map.mapId);

        alert('Map loaded successfully');

        if (map?.mapId) {
          var i;
          for (i = 0; i < 500; i++) {
            CapacitorGoogleMaps.addMarker({
              mapId: map.mapId,
              latitude: -this.getRandomArbitrary(33, 34),
              longitude: this.getRandomArbitrary(151, 152),
              title: 'Some title',
              snippet: 'Some snippet ',
              metadata: {
                some: 'data',
              },
            });
          }

          CapacitorGoogleMaps.didTapMarker(
            {
              mapId: map.mapId,
              preventDefault: false, // set to true if you do not want the default behaviour
            },
            (result) => {
              this.maps[refName].currentMarkerId = result.marker.id;
              alert(JSON.stringify(result));
            }
          );

          CapacitorGoogleMaps.didCloseInfoWindow(
            {
              mapId: map.mapId,
            },
            (result) => {
              // this will never fire, because it is overwritten by the next `didCloseInfoWindow`
              alert('this will never be shown');
            }
          );

          CapacitorGoogleMaps.didCloseInfoWindow(
            {
              mapId: map.mapId,
            },
            (result) => {
              // but this will fire
              this.maps[refName].currentMarkerId = null;
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
