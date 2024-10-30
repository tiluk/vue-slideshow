# Vue Slideshow

**This is a simple Vue with vuetify project that creates a slideshow from all images & videos in a directory.**

The slideshow will show all images and videos in the `static-assets/slideshow` directory in a random order.
The images will be shown for 5 seconds and the videos will be shown for their duration.
Touch gestures are supported for changing slides on mobile devices.
After the last image/video the slideshow will start again from the beginning with a new random order.

_It is currently not implemented to be usable as just a component you import_

_Due to problems with handling files in vite the dev mode isn't working_

## Usage

Place all images & videos you want to show in the slideshow in the `static-assets/slideshow` directory.
The component will automatically load all files in that directory and show them in the slideshow in a random order.

Then build the project with `npm run build` and run the dist folder with a server of your choice.

## Development

### Requirements

- Bun
- _The http server of your choosing_

Before you can start the development server you need to install the dependencies with `bun install`.

Build your project with `bun run build` and then start the http server of your choice in the `dist` directory.

### Technologies

- Vue
- Vuetify
- Typescript
- Vite
- vite-plugin-static-copy