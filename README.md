<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Upload and Display KMZ File</title>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.7.1/dist/leaflet.css" />
    <style>
        #map {
            height: 600px;
            width: 100%;
        }
    </style>
</head>
<body>
    <h1>Upload and Display KMZ File</h1>
    <form id="uploadForm">
        <label for="kmzFile">Choose a KMZ file:</label>
        <input type="file" id="kmzFile" name="kmzFile" accept=".kmz">
        <br><br>
        <input type="submit" value="Upload">
    </form>
    <div id="map"></div>

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
                        const geoJsonLayer = L.geoJSON(geojson).addTo(map);
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
        const kmzBase64 = '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';
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
                                const geoJsonLayer = L.geoJSON(geojson).addTo(map);
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
</body>
</html>
