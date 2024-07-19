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

        const loadKMZFromURL = (url) => {
            fetch(url)
                .then(response => {
                    if (!response.ok) {
                        throw new Error('Network response was not ok ' + response.statusText);
                    }
                    return response.arrayBuffer();
                })
                .then(data => {
                    JSZip.loadAsync(data).then(zip => {
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
                })
                .catch(error => {
                    console.error('Fetch error: ', error);
                });
        };

        // URL of your KMZ file on OneDrive
        const kmzFileUrl = 'https://onedrive.live.com/download?cid=33E2633D6589CBBC&id=33E2633D6589CBBC%21148755';
        loadKMZFromURL(kmzFileUrl);

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
