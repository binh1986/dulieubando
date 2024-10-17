<script src="https://unpkg.com/leaflet@1.7.1/dist/leaflet.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/togeojson/0.16.0/togeojson.min.js"></script>
<script src="https://unpkg.com/jszip@3.2.2/dist/jszip.min.js"></script>
<script src="https://unpkg.com/leaflet-filelayer/leaflet.filelayer.js"></script>

<script>
    const map = L.map('map').setView([0, 0], 2);

    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        maxZoom: 18,
        attribution: '© OpenStreetMap'
    }).addTo(map);

    const loadKMZFromBase64 = (base64String) => {
        const binaryString = atob(base64String);
        const len = binaryString.length;
        const bytes = new Uint8Array(len);
        for (let i = 0; i < len; i++) {
            bytes[i] = binaryString.charCodeAt(i);
        }
        JSZip.loadAsync(bytes.buffer).then(zip => {
            const kmlFile = zip.file(/\.kml$/i)[0];
            if (kmlFile) {
                kmlFile.async("string").then(kmlContent => {
                    const parser = new DOMParser();
                    const kmlDoc = parser.parseFromString(kmlContent, "text/xml");
                    const geojson = toGeoJSON.kml(kmlDoc);

                    const geoJsonLayer = L.geoJSON(geojson, {
                        onEachFeature: function (feature, layer) {
                            if (feature.properties && feature.properties.name) {
                                layer.bindPopup(`<b>${feature.properties.name}</b>`);
                            }
                        }
                    }).addTo(map);

                    map.fitBounds(geoJsonLayer.getBounds());
                });
            } else {
                console.error('No KML file found in the KMZ archive.');
            }
        }).catch(error => {
            console.error('Error reading KMZ file: ', error);
        });
    };

    // Replace with your own Base64 string
    const kmzBase64 = 'UEsDBBQAAgAIAAAAIABKewfXEgUAAG0VAAAkAAAARDgzNTlDMjUyQUUxNEJBMkI4OTU5OUU2QTQxQzYyMTcueHNs5Vhrb9s2...'; // shortened for readability
    loadKMZFromBase64(kmzBase64);

    document.getElementById('uploadForm').addEventListener('submit', function(event) {
        event.preventDefault();
        const fileInput = document.getElementById('kmzFile');
        const file = fileInput.files[0];
        
        if (file && file.name.endsWith('.kmz')) {
            const reader = new FileReader();
            reader.onload = function(e) {
                JSZip.loadAsync(e.target.result).then(zip => {
                    const kmlFile = zip.file(/\.kml$/i)[0];
                    if (kmlFile) {
                        kmlFile.async("string").then(kmlContent => {
                            const parser = new DOMParser();
                            const kmlDoc = parser.parseFromString(kmlContent, "text/xml");
                            const geojson = toGeoJSON.kml(kmlDoc);

                            const geoJsonLayer = L.geoJSON(geojson, {
                                onEachFeature: function (feature, layer) {
                                    if (feature.properties && feature.properties.name) {
                                        layer.bindPopup(`<b>${feature.properties.name}</b>`);
                                    }
                                }
                            }).addTo(map);

                            map.fitBounds(geoJsonLayer.getBounds());
                        });
                    } else {
                        alert('No KML file found in the KMZ archive.');
                    }
                }).catch(error => {
                    console.error('Error reading KMZ file: ', error);
                });
            };
            reader.readAsArrayBuffer(file);
        } else {
            alert('Please upload a valid KMZ file.');
        }
    });
</script>
