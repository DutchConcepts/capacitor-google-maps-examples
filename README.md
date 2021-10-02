# Google Maps for Capacitor Examples

## How to Run

1. Change into the UI Framework directory of choice. (e.g. `cd vue3`)
1. Run `npm install`.
1. Build the web app: `npm run build`.
1. Copy the built web app into the native platform: `npx cap copy`.
1. Make sure you've added your API-keys. Replace the occurrences of `your_api_key` with your API-key.
  - Android: In `AndroidManifest.xml` look for `your_api_key`.
  - iOS: in the `src` folder (within the UI Framework directory of choice) search for `your_api_key`.
  - For info about obtaining API-keys: https://github.com/DutchConcepts/capacitor-google-maps/tree/next#obtain-api-keys
1. Open the native IDE and run the app: `npx cap open android` or `npx cap open ios`.
