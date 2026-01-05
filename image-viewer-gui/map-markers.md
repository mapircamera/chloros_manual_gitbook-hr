# Map Markers

The Map tab displays your images on an interactive 2D map based on their GPS coordinates. This provides a geographic overview of your capture session and helps you visualize spatial coverage. It is also useful when first importing your images to quickly remove any images you do not need to process.

## Accessing the Map Tab

1. Open or create a project in Chloros
2. Import images that contain GPS metadata
3. Click the **Map** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> tab in the left sidebar
4. The map will display markers at each image's GPS location

{% hint style="info" %}
**GPS Required**: Only images with embedded GPS coordinates in their EXIF metadata will appear on the map. Ensure your camera has GPS enabled during capture.
{% endhint %}

***

## Adjusting Images from Map Tab

The **Map** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> tab has the same add  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  and remove  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  file buttons as the [**File Browser**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> tab does. It also shows the same project file table list but with different column headers:

### File Name

* Original filename from the camera
* Maintains camera naming convention (e.g., IMG\_0001.RAW)

### Latitude

* The image's latitude

### Longitude

* The image's longitude

### Altitude

* The image's altitude

{% hint style="info" %}
Clicking the table column headers also sorts the row data
{% endhint %}

***

## Image Markers

Each image with GPS data is represented by a marker on the map:

### Marker Display

* Markers indicate the exact GPS coordinates where each image was captured
* Clustered markers may group together when zoomed out
* Zoom in to see individual image locations

{% hint style="success" %}
SUPER-ZOOM: When you reach the maximum zoom level from the map tile provider the tile is then enlarged upon further zoom, allowing you to see markers that are close together.
{% endhint %}

### Hover Preview

* **Hover your mouse** over any marker to see a thumbnail preview of that image
* This allows quick visual identification without leaving the map view
* Useful for locating specific images within a large capture session

***

## Map Tile Providers

{% hint style="success" %}
**Automatic Selection**: Chloros automatically chooses the tile service that provides the best zoom level for your current map location. You can manually switch between providers if desired.
{% endhint %}

The Map tab supports two tile providers for the background map imagery:

### Google Maps

* Standard satellite and map imagery from Google
* Best for general worldwide coverage

### ESRI

* Satellite and aerial imagery from ESRI ArcGIS
* Often provides higher resolution imagery in certain regions

***

## Map Tile Types

You can choose the map layer type (from left to right):

&#x20;<img src="../.gitbook/assets/image (23).png" alt="" data-size="original">

### Terrain

Shows elevation profiles and map tiles with details (roads, etc)

### Map

Shows standard (lower bandwidth) map tiles with details (roads, etc)

### Satellite

Shows detailed (higher bandwidth) satellite map tiles

### Hybrid

Shows satellite map tiles with added details (roads, etc)

***

## Map Navigation

### Zoom Controls

* **Zoom In/Out**: Use mouse scroll wheel or zoom buttons
* **Fullscreen**: Fullscreen the map

### Pan Controls

* **Pan**: Click and drag to move around the map

***

## Use Cases

### Flight Path Visualization

* View the coverage area of drone capture sessions
* Identify gaps in image coverage
* Verify flight path execution

### Ground Survey Review

* See the spatial distribution of ground-based captures
* Locate calibration target images relative to survey area
* Plan additional capture locations

### Quality Control

* Quickly identify images captured in unexpected locations
* Verify GPS accuracy across the dataset
* Cross-reference image locations with field notes

***

## Troubleshooting

### No Markers Appearing

**Possible causes:**

* Images do not contain GPS metadata
* GPS was disabled on camera during capture
* EXIF data was stripped by external software

**Solution**: Verify GPS is enabled on your camera and re-import original files

### Markers in Wrong Location

**Possible causes:**

* Camera GPS had poor satellite fix
* GPS drift during capture

**Solution**: This is typically a capture-time issue; consider using PPK/RTK GPS for precision applications
