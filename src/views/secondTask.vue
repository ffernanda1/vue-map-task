<template>
  <div id="map" style="width: 100%; height: 400px;"></div>
</template>

<script>
import 'ol/ol.css';
import { Map, View } from 'ol';
import TileLayer from 'ol/layer/Tile';
import OSM from 'ol/source/OSM';
import { fromLonLat } from 'ol/proj';
import Feature from 'ol/Feature';
import Point from 'ol/geom/Point';
import { Style, Icon } from 'ol/style';
import VectorSource from 'ol/source/Vector';
import VectorLayer from 'ol/layer/Vector';

import locations from './location.json'; // Mengimpor data JSON

export default {
  mounted() {
    this.initMap();
    this.addMarkers();
  },
  methods: {
    initMap() {
      const map = new Map({
        target: 'map',
        layers: [
          new TileLayer({
            source: new OSM(),
          }),
        ],
        view: new View({
          center: fromLonLat([110.3658,-7.7864]),
          zoom: 10,
        }),
      });
      this.map = map;
    },
    addMarkers() {
      const vectorSource = new VectorSource();
      
      locations.forEach((location) => {
        const feature = new Feature({
          geometry: new Point(fromLonLat([location.longitude, location.latitude])),
        });

        const iconStyle = new Style({
          image: new Icon({
            anchor: [0.5, 1],
            src: 'https://openlayers.org/en/latest/examples/data/icon.png',
          }),
        });
        
        feature.setStyle(iconStyle);
        vectorSource.addFeature(feature);
      });

      const vectorLayer = new VectorLayer({
        source: vectorSource,
      });

      this.map.addLayer(vectorLayer);
    },
  },
};
</script>

<style scoped>
#map {
  width: 100%;
  height: 400px;
}
</style>
