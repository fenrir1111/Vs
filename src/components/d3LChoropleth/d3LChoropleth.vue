<template>
    <div class="d3-l-choropleth" :style="{ 'width' : width, 'height': height }"></div>
</template>

<script>
    import L from 'leaflet';
    import LChoropleth from 'leaflet-choropleth';
    import LIndoor from '../../lib/leaflet.indoor';
    import LFullscreen from 'leaflet-fullscreen';
    import mixins from '../../mixins/geoJson';

    export default {
        name: 'd3-l-choropleth',
        data() {
            return {
                _map: null,
                _tileLayer: null,
                _indoorLayer: null,
                _choroplethLayer: null,
                _fullscreenControl: null,
                _legend: null
            }
        },
        mixins: [mixins],
        methods: {
            drawLChoropleth() {
                const data = this.data,
                    indoorMaps = this.indoorMaps,
                    {
                        zoom = 18,
                        center,
                        url,
                        attribution,
                        maxZoom = 23,
                        minZoom = 1,

                        valueProperty = 'value',
                        scale = ['white', 'red'],
                        steps = 5,
                        mode = 'q',

                        color = '#fff',
                        weight = 2,
                        fillOpacity = 0.8
                    } = this.options;

                const container = document.createElement('div');
                container.style.width = '100%';
                container.style.height = '100%';

                const Map = this._map = L.map(this.$el.appendChild(container)).setView(center, zoom);

                this._tileLayer = L.tileLayer(url, {
                    attribution,
                    maxZoom,
                    minZoom
                }).addTo(Map);

                if (L.Util.isArray(indoorMaps) && indoorMaps.length > 0) {
                    this._indoorLayer = L.indoorLayer(indoorMaps, {
                        grayscale: true
                    }).addTo(Map);
                }

                this._choroplethLayer = L.choropleth(data, {
                    valueProperty,
                    scale,
                    steps,
                    mode,
                    style: {
                        color,
                        weight,
                        fillOpacity
                    },
                    onEachFeature: function (feature, layer) {
                        layer.bindPopup(`Zone: ${feature.properties.name}<br>Value: ${feature.properties.value.toFixed(2)}`);
                    }
                }).addTo(Map);

                const legend = this._legend = L.control({
                    position: 'topright'
                });

                const self = this;
                legend.onAdd = function (map) {
                    const div = L.DomUtil.create('div', 'info legend');
                    const limits = self._choroplethLayer.options.limits;
                    const colors = self._choroplethLayer.options.colors;
                    const labels = [];

                    div.innerHTML = '<div class="labels"><div class="min">' + limits[0].toFixed(2) + '</div> \
                        <div class="max">' + limits[limits.length - 1].toFixed(2) + '</div></div>';

                    limits.forEach(function (limit, index) {
                        labels.push('<li style="background-color: ' + colors[index] + '"></li>')
                    });

                    div.innerHTML += '<ul>' + labels.join('') + '</ul>';
                    return div
                };

                legend.addTo(Map);

                this._fullscreenControl = new L.Control.Fullscreen();
                Map.addControl(this._fullscreenControl);
            },
            reset() {
                this.$el.innerHTML = '';

                this._map = null;

                this._choroplethLayer = null;
                this._indoorLayer = null;
                this._fullscreenControl = null;
                this._tileLayer = null;
                this._legend = null;
            },
            safeDraw() {
                this.reset();
                this.drawLChoropleth();
            }
        }
    }
</script>

<style>
    @import url('../../css/index.css');

    .d3-l-choropleth .legend {
        color: #555;
        padding: 6px 8px;
        font: 12px Arial, Helvetica, sans-serif;
        font-weight: bold;
        background: white;
        background: rgba(255, 255, 255, 0.8);
        box-shadow: 0 0 15px rgba(0, 0, 0, 0.2);
        border-radius: 5px;
    }

     .d3-l-choropleth .legend ul {
        list-style-type: none;
        padding: 0px;
        margin: 0px;
        clear: both;
    }

    .d3-l-choropleth .legend li {
        display: inline-block;
        width: 30px;
        height: 22px;
    }

    .d3-l-choropleth .legend .min {
        float: left;
        padding-bottom: 5px;
    }

    .d3-l-choropleth .legend .max {
        float: right;
    }
</style>


