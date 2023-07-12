<template>
    <div>
      <div id="map" style="width: 100%; height: 400px;"></div>
      <div>
        <p>Koordinat: {{ markerCoordinate }}</p>
      </div>
    </div>
  </template>
  
  <script>
  import 'ol/ol.css';
  import { Map, View } from 'ol';
  import TileLayer from 'ol/layer/Tile';
  import OSM from 'ol/source/OSM';
  import { fromLonLat, toLonLat } from 'ol/proj';
  import { Feature } from 'ol';
  import { Point } from 'ol/geom';
  import  VectorLayer  from 'ol/layer/Vector';
  import  VectorSource  from 'ol/source/Vector';
  import { Modify, Snap } from 'ol/interaction';
  import { Style, Icon } from 'ol/style';
  
  export default {
    mounted() {
      this.initMap();
    },
    data() {
      return {
        map: null,
        markerFeature: null,
        markerCoordinate: '',
      };
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
            center: fromLonLat([110.3695, -7.7956]),
            zoom: 10,
          }),
        });
  
        const markerSource = new VectorSource();
  
        this.markerFeature = new Feature({
          geometry: new Point(fromLonLat([110.3695, -7.7956])),
        });
  
        markerSource.addFeature(this.markerFeature);
  
        const markerLayer = new VectorLayer({
          source: markerSource,
        });
  
        this.map = map;
        this.map.addLayer(markerLayer);
  
        const modifyInteraction = new Modify({
          features: new VectorSource({ features: [this.markerFeature] }),
          pixelTolerance: 100,
        });
  
        const snapInteraction = new Snap({
          source: markerSource,
        });
  
        this.map.addInteraction(modifyInteraction);
        this.map.addInteraction(snapInteraction);
  
        this.map.on('pointermove', (event) => {
          const coordinate = toLonLat(event.coordinate).map((coord) => coord.toFixed(6));
          this.markerCoordinate = coordinate.join(',');
        });
  
        this.map.on('click', () => {
          this.markerCoordinate = '';
        });
  
        const iconStyle = new Style({
          image: new Icon({
            anchor: [0.5, 1],
            src: 'https://openlayers.org/en/latest/examples/data/icon.png',
          }),
        });
  
       markerFeature.setStyle(iconStyle);
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
  