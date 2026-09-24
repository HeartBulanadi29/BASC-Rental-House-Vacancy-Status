<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Rental Vacancy Status Map</title>

  <!-- Leaflet CSS -->
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
  
  <style>
    body {
      margin: 0;
      padding: 0;
      font-family: Arial, sans-serif;
    }
    #map {
      width: 100vw;
      height: 100vh;
    }
    /* Custom Popup Styling */
    .popup-title {
      font-weight: bold;
      font-size: 14px;
      margin-bottom: 5px;
    }
    .status-badge {
      display: inline-block;
      padding: 2px 6px;
      border-radius: 4px;
      color: #fff;
      font-size: 11px;
      font-weight: bold;
    }
    .status-vacant { background-color: #28a745; }
    .status-occupied { background-color: #dc3545; }
  </style>
</head>
<body>

  <div id="map"></div>

  <!-- Leaflet JS -->
  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
  <!-- PapaParse JS (for parsing Google Sheet CSV directly) -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.3.2/papaparse.min.js"></script>

  <script>
    // 1. Initialize Map
    const map = L.map('map').setView([15.034291, 121.025233], 14); // Adjust initial center [Latitude, Longitude] and zoom level

    // 2. Add OpenStreetMap Base Layer
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
  maxZoom: 19,
  attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
}).addTo(map);

    // 3. Define Custom Colored Marker Icons
    const greenIcon = L.icon({
      iconUrl: 'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-green.png',
      shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png',
      iconSize: [25, 41],
      iconAnchor: [12, 41],
      popupAnchor: [1, -34],
      shadowSize: [41, 41]
    });

    const redIcon = L.icon({
      iconUrl: 'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-red.png',
      shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png',
      iconSize: [25, 41],
      iconAnchor: [12, 41],
      popupAnchor: [1, -34],
      shadowSize: [41, 41]
    });

    // 4. Link to Published Google Sheets CSV URL
    const csvUrl = 'https://docs.google.com/spreadsheets/d/e/2PACX-1vS205Y5MwdIs1_lBAedenvsgzdv9jNhKzvdFv9odBLMMqy2fhfE7GqkUQDdBQs0TK2LWlljqdL_n183/pub?gid=77072596&single=true&output=csv';

    // 5. Fetch and Parse CSV Data Live
    Papa.parse(csvUrl, {
      download: true,
      header: true,
      complete: function(results) {
        const data = results.data;

        data.forEach(row => {
          const lat = parseFloat(row['Latitude']);
          const lng = parseFloat(row['Longitude']);
          const status = row['Status'] ? row['Status'].trim() : '';

          if (!isNaN(lat) && !isNaN(lng)) {
            // Select marker color based on status
            const markerIcon = (status.toLowerCase() === 'vacant') ? greenIcon : redIcon;
            const badgeClass = (status.toLowerCase() === 'vacant') ? 'status-vacant' : 'status-occupied';

            // Custom Popup panel displaying sheet properties
            const popupContent = `
             // Custom Popup panel displaying sheet properties
const popupContent = `
  <div class="popup-title">${row['BASC Vacancy Status'] || 'BASC Vacancy Status'}</div>
  <div><span class="status-badge ${badgeClass}">${status?.toUpperCase() || 'N/A'}</span></div>
  <hr style="margin: 8px 0; border: 0; border-top: 1px solid #ccc;">
  <p style="margin: 4px 0;"><b>Email:</b> ${row['Email'] || 'N/A'}</p>
  <p style="margin: 4px 0;"><b>Contact Number:</b> ${row['Contact Number'] || 'N/A'}</p>
  <p style="margin: 4px 0;"><b>Rental House Name:</b> ${row['Rental House Name'] || 'N/A'}</p>
  <p style="margin: 4px 0;"><b>Type of Rental Housing:</b> ${row['Type of Rental House'] || 'N/A'}</p>
  <p style="margin: 4px 0;"><b>Current Available Status:</b> ${row['Current Availability Status'] || 'N/A'}</p>
  <p style="margin: 4px 0;"><b>Number of Available Rooms:</b> ${row['Number of Available Rooms'] || 'N/A'}</p>
  <p style="margin: 4px 0;"><b>Number of Available Beds:</b> ${row['Number of Available Beds'] || 'N/A'}</p>
`;
            `;

            // Place marker on map with popup panel
            L.marker([lat, lng], { icon: markerIcon })
              .bindPopup(popupContent)
              .addTo(map);
          }
        });
      }
    });
  </script>
</body>
</html>
