<template>
    <div>
        <div id="map" style="width: 100%; height: 400px;"></div>
        <div v-if="selectedCluster">
            <h2>Lokasi yang Digabungkan</h2>
            <ul>
                <li v-for="location in selectedCluster" :key="location.name">{{ location.name }}</li>
            </ul>
        </div>
    </div>
</template>
  
<script>
import 'ol/ol.css';
import Point from 'ol/geom/Point';
import { Map, View } from 'ol';
import TileLayer from 'ol/layer/Tile';
import OSM from 'ol/source/OSM';
import { fromLonLat } from 'ol/proj';
import Cluster from 'ol/source/Cluster';
import { Style, Text, Circle, Fill, Stroke } from 'ol/style';
import VectorLayer from 'ol/layer/Vector';
import VectorSource from 'ol/source/Vector';
import Select from 'ol/interaction/Select';
import Feature from 'ol/Feature';
import * as conditions from 'ol/events/condition';


import locations from './location.json';

export default {
    mounted() {
        this.initMap();
    },
    data() {
        return {
            map: null,
            selectedCluster: [],
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

            const features = locations.map((location) => {

                return new Feature({
                    geometry: new Point(fromLonLat([location.longitude, location.latitude])),
                    name: location.name,
                });
            });

            const clusterSource = new Cluster({
                distance: 40,
                source: new VectorSource({
                    features: features,
                }),
            });

            const clusterStyle = new Style({
                image: new Circle({
                    radius: 10,
                    fill: new Fill({ color: 'rgba(0, 0, 255, 0.1)' }),
                    stroke: new Stroke({ color: 'blue', width: 1 }),
                }),
                text: new Text({
                    text: '',
                    fill: new Fill({ color: 'blue' }),
                }),
            });

            const clusterLayer = new VectorLayer({
                source: clusterSource,
                style: function (feature) {
                    const size = feature.get('features').length;
                    const textStyle = clusterStyle.getText();
                    textStyle.setText(size.toString());
                    return clusterStyle;
                },
            });

            map.addLayer(clusterLayer);

            const selectInteraction = new Select({
                condition: conditions.click,
                layers: [clusterLayer],
            });

            selectInteraction.on('select', (event) => {
                const selectedFeatures = event.target.getFeatures().getArray();
                if (selectedFeatures.length === 1 && selectedFeatures[0].get('features').length > 1) {
                    const clusterFeatures = selectedFeatures[0].get('features');
                    const locationNames = clusterFeatures.map((feature) => feature.get("name"));
                    this.selectedCluster = [...locationNames];
                } else {
                    this.selectedCluster = null;
                }
            });
            map.addInteraction(selectInteraction);

            this.map = map;
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
  