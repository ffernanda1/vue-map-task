<template>
  <div>
    <div ref="map" class="map"></div>
    <div class="coordinates">
      <h3>Marker Coordinates:</h3>
      <p>{{ markerCoordinates }}</p>
    </div>
  </div>
</template>

<script>
import 'ol/ol.css';
import { Map, View } from 'ol';
import { fromLonLat, toLonLat } from 'ol/proj';
import  DragAndDrop  from 'ol/interaction/DragAndDrop.js';
import { Tile as TileLayer, Vector as VectorLayer } from 'ol/layer';
import { OSM, Vector as VectorSource } from 'ol/source';
import { Icon, Style } from 'ol/style';
import Point from 'ol/geom/Point';
import Feature from 'ol/Feature';
import GeoJSON from 'ol/format/GeoJSON.js';
import GPX from 'ol/format/GPX.js';
import TopoJSON from 'ol/format/TopoJSON.js';
import IGC from 'ol/format/IGC.js';
import KML from 'ol/format/KML.js';

export default {
  data() {
    return {
      map: null,
      marker: null,
      markerCoordinates: '',
    };
  },
  mounted() {
    this.initializeMap();
  },
  methods: {
    initializeMap() {
      const centerCoordinates = fromLonLat([ 110.3695,-7.7956]);

      this.map = new Map({
        target: this.$refs.map,
        layers: [
          new TileLayer({
            source: new OSM(),
          }),
        ],
        view: new View({
          center: centerCoordinates,
          zoom: 10,
        }),
      });

      this.addMarker(centerCoordinates);
      this.addDragAndDropInteraction();

      console.log(this.addDragAndDropInteraction());
    },
    addMarker(coordinates) {
      const iconStyle = new Style({
        image: new Icon({
          anchor: [0.5, 1],
          src: 'https://openlayers.org/en/latest/examples/data/icon.png',
        }),
      });

      const markerSource = new VectorSource({
        features: [
          new Feature({
            geometry: new Point(coordinates),
          }),
        ],
      });

      const markerLayer = new VectorLayer({
        source: markerSource,
        style: iconStyle,
      });

      this.map.addLayer(markerLayer);
      this.marker = markerSource.getFeatures()[0];
    },
    addDragAndDropInteraction() {
      const dragAndDropInteraction = new DragAndDrop({
        formatConstructors: [GPX, GeoJSON, IGC, KML, TopoJSON],
      });

      dragAndDropInteraction.on('addfeatures', (event) => {
        const coordinate = event.features[0].getGeometry().getCoordinates();
        const lonLatCoordinate = toLonLat(coordinate);
        this.updateMarkerCoordinates(lonLatCoordinate);
      });

      this.map.addInteraction(dragAndDropInteraction);
    },
    updateMarkerCoordinates(coordinates) {
      const lon = coordinates[0].toFixed(4);
      const lat = coordinates[1].toFixed(4);
      this.markerCoordinates = `${lat}, ${lon}`;
    },
  },
};
</script>

<style scoped>
.map {
  width: 100%;
  height: 400px;
}

.coordinates {
  margin-top: 20px;
}
</style>
