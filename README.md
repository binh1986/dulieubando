<script src="https://unpkg.com/leaflet@1.7.1/dist/leaflet.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/togeojson/0.16.0/togeojson.min.js"></script>
<script src="https://unpkg.com/jszip@3.2.2/dist/jszip.min.js"></script>
<script src="https://unpkg.com/leaflet-filelayer/leaflet.filelayer.js"></script>
<script>
    const map = L.map('map', {
        maxZoom: 21, // Đặt mức phóng to tối đa lên 21
        zoomControl: true // Bật điều khiển zoom
    }).setView([0, 0], 2);

    // Sử dụng lớp ảnh vệ tinh của ESRI với zoom tối đa
    L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}', {
        maxZoom: 21, // Đảm bảo lớp này cũng hỗ trợ mức zoom tối đa là 21
        attribution: '© Esri'
    }).addTo(map);

    // Xử lý việc tải file KMZ và hiển thị lên bản đồ
    document.getElementById('uploadForm').addEventListener('submit', function(event) {
        event.preventDefault();
        const fileInput = document.getElementById('kmzFile');
        const file = fileInput.files[0];
        
        if (file && file.name.endsWith('.kmz')) {
            const reader = new FileReader();
            reader.onload = function(e) {
                JSZip.loadAsync(e.target.result).then(function(zip) {
                    const kmlFile = zip.file(/\.kml$/i)[0];
                    if (kmlFile) {
                        kmlFile.async("string").then(function(kmlContent) {
                            const parser = new DOMParser();
                            const kmlDoc = parser.parseFromString(kmlContent, "text/xml");
                            const geojson = toGeoJSON.kml(kmlDoc);

                            const geoJsonLayer = L.geoJSON(geojson, {
                                onEachFeature: function (feature, layer) {
                                    let popupContent = "<b>Details:</b><br>";
                                    if (feature.properties) {
                                        for (let key in feature.properties) {
                                            popupContent += `${key}: ${feature.properties[key]}<br>`;
                                        }
                                    }
                                    layer.bindPopup(popupContent);
                                }
                            }).addTo(map);

                            map.fitBounds(geoJsonLayer.getBounds());
                        });
                    } else {
                        alert('No KML file found in the KMZ archive.');
                    }
                });
            };
            reader.readAsArrayBuffer(file);
        } else {
            alert('Please upload a valid KMZ file.');
        }
    });
</script>
