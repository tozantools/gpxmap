# GPX Gradient Visualizer

## Overview

A Leaflet-based web application that visualizes GPX track data with color-coded gradient information based on elevation changes. The tool provides an interactive map interface where users can upload GPX files to analyze terrain difficulty through visual gradient representation.

## Core Functionality

### Map Visualization
- **Base Map**: Uses Japan's Geospatial Information Authority (GSI) tile layers
- **Layer Options**:
  - Standard map (`std` layer)
  - Hillshade map (`hillshademap` layer) for topographic relief visualization
- **Interactive Controls**:
  - Layer switcher for toggling between map styles
  - Scale control for distance reference
  - Custom attribution display in top-right corner

### URL Parameter Support
The application supports deep-linking with URL parameters for initial map view:
- `c`: Center coordinates (comma-separated lat,lng)
- `z`: Zoom level
- **Default view**: Japan center (36.3428, 137.6478) at zoom level 6
- **Example**: `?c=35.6762,139.6503&z=12` (Tokyo, zoom 12)

### GPX File Input Methods
Users can load GPX files through two methods:
1. **File Upload**: Traditional file input button (bottom-left corner)
2. **Drag & Drop**: Direct file drop onto the map canvas

### Gradient Visualization

#### Data Processing Pipeline
1. **GPX Parsing** (lines 74-83):
   - Extracts trackpoint elements (`<trkpt>`)
   - Parses latitude, longitude, and elevation data
   - Builds array of [lat, lon, elevation] tuples

2. **Auto-fit View** (lines 89-92):
   - Calculates bounding box of entire track
   - Automatically adjusts map view to show full route

3. **Gradient Calculation** (lines 122-139):
   - Processes consecutive trackpoint pairs
   - Calculates horizontal distance using haversine formula
   - Computes elevation change between points
   - Determines gradient angle in degrees: `arctan(Δelevation / distance)`

4. **Color Mapping** (lines 141-145):
   - **Red**: Gradients > 30° (extremely steep)
   - **Yellow**: Gradients 15-30° (moderately steep)
   - **Green**: Gradients < 15° (gentle/flat)

#### Rendering
- Each segment between consecutive trackpoints rendered as individual polyline
- Color applied based on gradient calculation
- Creates smooth visual flow showing terrain difficulty

## Technical Implementation

### Distance Calculation Methods

The application includes three geodesic distance calculation implementations:

#### 1. Haversine Formula (Active) (lines 147-160)
```javascript
getDistance(lat1, lon1, lat2, lon2)
```
- **Current default method**
- Assumes spherical Earth model (radius: 6,371 km)
- Fast computation, sufficient for most use cases
- Accuracy: ±0.5% for distances up to several hundred kilometers

#### 2. Vincenty Formula (Available, Commented) (lines 162-207)
```javascript
getDistanceVincenty(lat1, lon1, lat2, lon2)
```
- Uses WGS-84 ellipsoid model
- More accurate for long distances and near poles
- Iterative convergence algorithm (max 100 iterations)
- **Parameters**:
  - Semi-major axis: 6,378,137 m
  - Flattening: 1/298.257223563
- Handles edge cases (equatorial lines, identical points)

#### 3. Karney's Algorithm (Available, Commented) (line 129)
- External library: `geographiclib-geodesic`
- Most accurate geodesic calculation
- Numerically stable for all distance ranges
- Currently commented out in both dependency (line 9) and usage

### Key Functions

#### `getMapParams()` (lines 40-48)
- Parses URL query parameters
- Returns object with `coords` and `zoom` properties
- Provides fallback defaults if parameters missing

#### `handleFileUpload(file)` (lines 71-103)
- Core file processing function
- Uses `FileReader` API for asynchronous file reading
- XML parsing via `DOMParser`
- Orchestrates visualization pipeline

#### `calculateGradients(data)` (lines 122-139)
- Main gradient computation engine
- Returns array of segment objects containing:
  - `start`: Starting point [lat, lon, ele]
  - `end`: Ending point [lat, lon, ele]
  - `gradient`: Calculated angle in degrees

#### `getColorForGradient(gradient)` (lines 141-145)
- Gradient-to-color mapper
- Uses absolute value for bidirectional gradients (uphill/downhill)
- Returns CSS color strings

### Event Handling

#### File Input (lines 106-108)
- Standard `change` event on file input element
- Passes selected file to `handleFileUpload()`

#### Drag & Drop (lines 111-120)
- `dragover` event: Prevents default browser behavior
- `drop` event:
  - Extracts file from `dataTransfer`
  - Validates MIME type (`application/gpx+xml`)
  - Processes valid GPX files

## Architecture Patterns

### Single-File Architecture
- Completely self-contained HTML document
- No external JavaScript dependencies beyond Leaflet library
- Inline CSS and JavaScript for portability

### Data Flow
```
GPX File Input
    ↓
File Reader (Async)
    ↓
XML Parsing
    ↓
Trackpoint Extraction
    ↓
Gradient Calculation
    ↓
Color Mapping
    ↓
Polyline Rendering
    ↓
Map Display
```

### Error Handling
- Limited explicit error handling
- Relies on browser's native error reporting
- Drag & drop validates file MIME type
- Distance calculation handles edge cases (identical points, equatorial lines)

## Dependencies

### External Libraries
- **Leaflet 1.9.4**: Core mapping library
  - CSS: Map styling and controls
  - JS: Map manipulation and layer management
- **国土地理院 Tile Layers**: Japanese government map data
  - Standard map tiles
  - Hillshade relief tiles

### Optional Dependencies (Commented)
- **geographiclib-geodesic 2.0.0**: Advanced geodesic calculations

## Browser Compatibility

### Required APIs
- `FileReader` API (file upload)
- `DOMParser` API (XML parsing)
- `URLSearchParams` API (URL parameter parsing)
- `DataTransfer` API (drag & drop)
- ES6 features:
  - Arrow functions
  - Template literals
  - Destructuring assignment
  - `const`/`let` declarations

### Supported Browsers
- Chrome/Edge 49+
- Firefox 44+
- Safari 10.1+
- Opera 36+

## Performance Considerations

### Scalability
- Renders one polyline per segment pair
- Large GPX files (thousands of points) may impact performance
- No optimization for segment batching or simplification

### Potential Bottlenecks
1. **DOM manipulation**: Creating individual polyline elements for each segment
2. **Synchronous rendering**: All segments rendered in single loop
3. **Memory usage**: Stores full trackpoint array in memory

### Optimization Opportunities
- Implement track simplification (Douglas-Peucker algorithm)
- Use canvas renderer for large datasets
- Batch polyline creation
- Implement web workers for gradient calculation

## Usage Examples

### Basic Usage
1. Open `index.html` in browser
2. Click file input button or drag GPX file onto map
3. View color-coded gradient visualization

### URL Parameters
```
# Tokyo city center at street level
index.html?c=35.6812,139.7671&z=14

# Mount Fuji area overview
index.html?c=35.3606,138.7274&z=11

# Default Japan view
index.html
```

### Expected GPX Structure
```xml
<?xml version="1.0"?>
<gpx version="1.1">
  <trk>
    <trkseg>
      <trkpt lat="35.6812" lon="139.7671">
        <ele>40.5</ele>
      </trkpt>
      <trkpt lat="35.6815" lon="139.7675">
        <ele>42.3</ele>
      </trkpt>
      <!-- More trackpoints -->
    </trkseg>
  </trk>
</gpx>
```

## Limitations

1. **Single Track Support**: Only processes first track segment
2. **No Time Data**: Ignores timestamp information if present
3. **Fixed Color Scheme**: Hardcoded gradient thresholds and colors
4. **No Export**: Cannot save or export visualization
5. **Client-Side Only**: No server-side processing or storage
6. **Language**: UI elements and comments in Japanese

## Future Enhancement Opportunities

### Features
- Multiple track support
- Customizable gradient thresholds and color schemes
- Speed calculation using time data
- Elevation profile chart
- Statistics panel (total distance, elevation gain/loss, max gradient)
- Track editing capabilities
- Export visualization as image or data

### Technical Improvements
- Internationalization (i18n) support
- Progressive Web App (PWA) capabilities
- Offline mode with service workers
- Performance optimization for large files
- Unit tests for distance calculations
- TypeScript conversion for type safety

### UX Enhancements
- Progress indicator during file processing
- Error messages for invalid GPX files
- Legend showing gradient color meanings
- Tooltips on segment hover showing exact gradient
- Comparison mode for multiple tracks

## Code Quality Notes

### Strengths
- Clear function naming and structure
- Self-contained and portable
- Multiple distance calculation methods available
- Responsive viewport-based sizing

### Areas for Improvement
- No error handling for malformed GPX files
- Mixed language comments (Japanese)
- Magic numbers for gradient thresholds
- No input validation for elevation data
- Limited code documentation
- No separation of concerns (HTML/CSS/JS in single file)

## Security Considerations

### Current Security Posture
- Client-side only processing (no data transmission)
- File validation limited to MIME type check
- No sanitization of GPX content
- External CDN dependencies (Leaflet)

### Potential Vulnerabilities
- XML External Entity (XXE) injection via malicious GPX
- Cross-Site Scripting (XSS) if GPX contains malicious content
- Denial of Service via extremely large GPX files

### Recommendations
- Implement XML sanitization
- Validate GPX schema structure
- Add file size limits
- Consider Content Security Policy (CSP) headers
- Use Subresource Integrity (SRI) for CDN resources

## Project Context

### Git Repository Status
- Current branch: `docs`
- Main branch: `main`
- Recent commits focus on:
  - External center/zoom setting functionality
  - Hillshade layer addition
  - Distance calculation improvements (Karney, Vincenty algorithms)

### Modified Files
- `index.html`: Working changes not yet committed

## Development Workflow

### Making Changes
1. Checkout appropriate branch
2. Edit `index.html` directly
3. Test in browser (no build process required)
4. Commit changes to git

### Testing
- Manual testing in browser
- Test with various GPX files
- Verify different distance calculation methods
- Check mobile responsiveness

### Deployment
- Single file deployment
- Can be hosted on any static web server
- No build or compilation step required
