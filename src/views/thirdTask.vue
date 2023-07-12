<template>
    <div>
        <div id="map" style="width: 100%; height: 400px;"></div>
        <div>
            <input v-model="coordinate" type="text" placeholder="Koordinat">
            <button @click="addMarker">Tambah Marker</button>
        </div>
    </div>
</template>
  
<script>
import 'ol/ol.css';
import { Map, View } from 'ol';
import TileLayer from 'ol/layer/Tile';
import OSM from 'ol/source/OSM';
import { fromLonLat } from 'ol/proj';
import { Feature } from 'ol';
import { Point } from 'ol/geom';
import { Vector as VectorLayer } from 'ol/layer';
import { Vector as VectorSource } from 'ol/source';
import { Style, Icon } from 'ol/style';

export default {
    mounted() {
        this.initMap();
    },
    data() {
        return {
            map: null,
            coordinate: '',
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
            this.map = map;
        },
        addMarker() {
            const [longitude, latitude] = this.coordinate.split(',').map(coord => parseFloat(coord.trim()));
            if (isNaN(longitude) || isNaN(latitude)) {
                alert('Koordinat tidak valid!');
                return;
            }

            const marker = new Feature({
                geometry: new Point(fromLonLat([longitude, latitude])),
            });

            const markerLayer = new VectorLayer({
                source: new VectorSource({
                    features: [marker],
                }),
            });

            const iconStyle = new Style({
                image: new Icon({
                    anchor: [0.5, 1],
                    src: 'https://openlayers.org/en/latest/examples/data/icon.png',
                }),
            });
            marker.setStyle(iconStyle);
            this.map.addLayer(markerLayer);
            this.map.getView().animate({
                center: fromLonLat([longitude, latitude]),
                zoom: 10,
                duration: 1000,
            });
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
  