<template>
  <div>
    <div id="map" class="map"></div>
    <div id="marker" class="marker" draggable="true">
      Drag me!!!
    </div>
  </div>
</template>

<script>
import 'ol/ol.css';
import { Map, View } from 'ol';
import TileLayer from 'ol/layer/Tile';
import OSM from 'ol/source/OSM';
import Overlay from 'ol/Overlay';
import Feature from 'ol/Feature';
import Point from 'ol/geom/Point';
import { fromLonLat } from 'ol/proj';
import {  Modify } from 'ol/interaction';
import { Vector as VectorLayer } from 'ol/layer';
import { Vector as VectorSource } from 'ol/source';

export default {
  mounted() {
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

    const source = new VectorSource();
    const vectorLayer = new VectorLayer({
      source: source,
    });
    map.addLayer(vectorLayer);

    const dragInteraction = new Modify({
      source: source,
    });
    map.addInteraction(dragInteraction);

    const marker = new Feature({
      geometry: new Point(fromLonLat([0, 0])),
    });
    source.addFeature(marker);

    const overlay = new Overlay({
      element: document.getElementById('marker'),
      positioning: 'bottom-center',
      stopEvent: false,
    });
    map.addOverlay(overlay);

    map.on('click', (event) => {
      const coordinate = event.coordinate;
      marker.getGeometry().setCoordinates(coordinate);
      overlay.setPosition(coordinate);
    });

   map.on('pointermove', (event) => {
       if (dragInteraction && dragInteraction.source) {
        const features = dragInteraction.getFeatures().getArray();

        if (map.hasFeatureAtPixel(event.pixel) || features.length > 0) {
          map.getViewport().style.cursor = 'pointer';
        } else {
          map.getViewport().style.cursor = 'default';
        }
      }
    });

  },
};
</script>

<style>
.map {
  width: 100%;
  height: 400px;
}

.marker {
  position: absolute;
  background-color: rgba(255, 6, 6, 0.8);
  padding: 10px 20px;
  border: 2px solid #ccc;
  border-radius: 4px;
}
</style>