# UFO Game using HeaderTrackr.js to control it

A Vue Component that can be used to play a game using head movement from your webcam

## Getting Started

This requires the package headtrackr.js https://github.com/auduno/headtrackr

### Prerequisites

Install HeadTrackr
```
npm install headtrackr
```

In your bootstrap.js file add
```
window.headtrackr = require('headtrackr')
```

Set up your app.js
```
Vue.component('headtrackr', require('./components/Headtrackr.vue'));
```

Download the component and add it to the resources/assets/js/components folder.
Add the following to your View
```
<headtrackr></headtrackr>   
```
   
