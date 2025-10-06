# Mars Rover Photos API - Usage Examples

Quick reference for common API queries.

## Base URLs

```bash
# NASA Official (requires API key)
BASE_URL="https://api.nasa.gov/mars-photos/api/v1"
API_KEY="DEMO_KEY"

# Heroku Community Mirror (no API key required, CORS enabled)
BASE_URL="https://mars-photos.herokuapp.com/api/v1"
```

## Photos Endpoints

### Get Photos by Sol

```bash
# Get Curiosity photos from sol 1000
curl "${BASE_URL}/rovers/curiosity/photos?sol=1000&api_key=${API_KEY}"

# Get photos with specific camera
curl "${BASE_URL}/rovers/curiosity/photos?sol=1000&camera=NAVCAM&api_key=${API_KEY}"

# Paginate results
curl "${BASE_URL}/rovers/curiosity/photos?sol=1000&page=2&api_key=${API_KEY}"
```

### Get Photos by Earth Date

```bash
# Get photos from specific Earth date
curl "${BASE_URL}/rovers/curiosity/photos?earth_date=2015-05-30&api_key=${API_KEY}"

# Filter by camera
curl "${BASE_URL}/rovers/perseverance/photos?earth_date=2024-01-15&camera=NAVCAM_LEFT&api_key=${API_KEY}"
```

### Get Latest Photos

```bash
# Get latest photos from Curiosity
curl "${BASE_URL}/rovers/curiosity/latest_photos?api_key=${API_KEY}"

# Get latest photos from specific camera
curl "${BASE_URL}/rovers/curiosity/latest_photos?camera=NAVCAM&api_key=${API_KEY}"
```

### Get Specific Photo

```bash
# Get photo by ID
curl "${BASE_URL}/photos/102693?api_key=${API_KEY}"
```

## Manifest Endpoints

### Get Mission Manifest

```bash
# Get complete mission manifest
curl "${BASE_URL}/manifests/curiosity?api_key=${API_KEY}"

# Pretty print with jq
curl "${BASE_URL}/manifests/curiosity?api_key=${API_KEY}" | jq '.'

# Get just the photo count
curl "${BASE_URL}/manifests/curiosity?api_key=${API_KEY}" | jq '.photo_manifest.total_photos'

# Get sols with photos
curl "${BASE_URL}/manifests/curiosity?api_key=${API_KEY}" | jq '.photo_manifest.photos[] | {sol, total_photos, cameras}'
```

## Rover Endpoints

### List All Rovers

```bash
# Get all rovers
curl "${BASE_URL}/rovers?api_key=${API_KEY}"

# Extract rover names
curl "${BASE_URL}/rovers?api_key=${API_KEY}" | jq '.rovers[].name'
```

### Get Specific Rover

```bash
# Get rover details
curl "${BASE_URL}/rovers/curiosity?api_key=${API_KEY}"

# Get rover status
curl "${BASE_URL}/rovers/curiosity?api_key=${API_KEY}" | jq '.rover.status'

# Get total photos
curl "${BASE_URL}/rovers/curiosity?api_key=${API_KEY}" | jq '.rover.total_photos'
```

## Advanced Queries with jq

### Find Photos from Multiple Sols

```bash
# Get photos from sols 1000-1005
for sol in {1000..1005}; do
  echo "Sol $sol:"
  curl -s "${BASE_URL}/rovers/curiosity/photos?sol=$sol&api_key=${API_KEY}" \
    | jq '.photos | length'
done
```

### Download All Photos from a Sol

```bash
# Download all photo URLs from sol 1000
curl -s "${BASE_URL}/rovers/curiosity/photos?sol=1000&api_key=${API_KEY}" \
  | jq -r '.photos[].img_src' \
  | while read url; do
      wget "$url"
    done
```

### Get Photo Count by Camera

```bash
# Count photos by camera for a specific sol
curl -s "${BASE_URL}/rovers/curiosity/photos?sol=1000&api_key=${API_KEY}" \
  | jq -r '.photos[].camera.name' \
  | sort | uniq -c
```

### Find Sols with Most Photos

```bash
# Get top 10 sols with most photos
curl -s "${BASE_URL}/manifests/curiosity?api_key=${API_KEY}" \
  | jq '.photo_manifest.photos | sort_by(.total_photos) | reverse | .[0:10] | .[] | {sol, total_photos}'
```

## JavaScript/Fetch Examples

### Basic Fetch

```javascript
const API_KEY = 'DEMO_KEY';
const BASE_URL = 'https://api.nasa.gov/mars-photos/api/v1';

async function getPhotos(rover, sol) {
  const response = await fetch(
    `${BASE_URL}/rovers/${rover}/photos?sol=${sol}&api_key=${API_KEY}`
  );
  const data = await response.json();
  return data.photos;
}

// Usage
const photos = await getPhotos('curiosity', 1000);
console.log(`Found ${photos.length} photos`);
```

### With Error Handling

```javascript
async function getMarsPhotos(rover, sol, camera = null) {
  try {
    let url = `${BASE_URL}/rovers/${rover}/photos?sol=${sol}&api_key=${API_KEY}`;
    if (camera) {
      url += `&camera=${camera}`;
    }

    const response = await fetch(url);

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }

    const data = await response.json();
    return data.photos;
  } catch (error) {
    console.error('Failed to fetch photos:', error);
    throw error;
  }
}

// Usage
try {
  const photos = await getMarsPhotos('curiosity', 1000, 'NAVCAM');
  photos.forEach(photo => {
    console.log(`${photo.camera.name}: ${photo.img_src}`);
  });
} catch (error) {
  console.error('Error:', error.message);
}
```

## Python Examples

### Using requests

```python
import requests

API_KEY = 'DEMO_KEY'
BASE_URL = 'https://api.nasa.gov/mars-photos/api/v1'

def get_photos(rover, sol, camera=None):
    params = {
        'sol': sol,
        'api_key': API_KEY
    }
    if camera:
        params['camera'] = camera

    response = requests.get(f'{BASE_URL}/rovers/{rover}/photos', params=params)
    response.raise_for_status()
    return response.json()['photos']

# Usage
photos = get_photos('curiosity', 1000, camera='NAVCAM')
print(f'Found {len(photos)} photos')

for photo in photos[:5]:  # First 5
    print(f"{photo['camera']['name']}: {photo['img_src']}")
```

### Download Images

```python
import requests
from pathlib import Path

def download_photos(rover, sol, output_dir='downloads'):
    photos = get_photos(rover, sol)
    output_path = Path(output_dir)
    output_path.mkdir(exist_ok=True)

    for i, photo in enumerate(photos):
        filename = f"{rover}_sol{sol}_{photo['camera']['name']}_{i}.jpg"
        filepath = output_path / filename

        print(f'Downloading {filename}...')
        img_response = requests.get(photo['img_src'])
        filepath.write_bytes(img_response.content)

    print(f'Downloaded {len(photos)} photos to {output_dir}/')

# Usage
download_photos('curiosity', 1000)
```

## Common Use Cases

### 1. Build a Photo Gallery

```bash
# Get random photos from different sols
curl -s "${BASE_URL}/rovers/curiosity/photos?sol=1000&api_key=${API_KEY}" \
  | jq -r '.photos[0:10] | .[] | "\(.camera.name): \(.img_src)"'
```

### 2. Track Mission Progress

```bash
# Get latest mission status
curl -s "${BASE_URL}/manifests/curiosity?api_key=${API_KEY}" \
  | jq '{
      name: .photo_manifest.name,
      status: .photo_manifest.status,
      latest_sol: .photo_manifest.max_sol,
      latest_date: .photo_manifest.max_date,
      total_photos: .photo_manifest.total_photos
    }'
```

### 3. Find Photos by Date Range

```bash
# Get photos for a week (requires multiple requests)
for date in 2024-01-{15..21}; do
  echo "Date: $date"
  curl -s "${BASE_URL}/rovers/curiosity/photos?earth_date=$date&api_key=${API_KEY}" \
    | jq '.photos | length'
done
```

### 4. Compare Camera Activity

```bash
# Get photos by all cameras on a specific sol
for camera in FHAZ RHAZ MAST NAVCAM; do
  count=$(curl -s "${BASE_URL}/rovers/curiosity/photos?sol=1000&camera=$camera&api_key=${API_KEY}" \
    | jq '.photos | length')
  echo "$camera: $count photos"
done
```

## Rate Limiting

Remember:
- **DEMO_KEY**: 50 requests/hour
- **Personal Key**: 1,000 requests/hour

Check headers for rate limit info:

```bash
curl -i "${BASE_URL}/rovers/curiosity/photos?sol=1000&api_key=${API_KEY}" \
  | grep -i "x-ratelimit"
```

## Error Handling Examples

### Handle Missing API Key

```bash
# This will return 403 error
curl "${BASE_URL}/rovers/curiosity/photos?sol=1000"

# Expected response:
# {
#   "error": {
#     "code": "API_KEY_MISSING",
#     "message": "No api_key was supplied..."
#   }
# }
```

### Handle Invalid Rover

```bash
curl "${BASE_URL}/rovers/invalid/photos?sol=1000&api_key=${API_KEY}"

# Expected response:
# {
#   "errors": "Invalid Rover Name"
# }
```

### Handle Invalid Sol

```bash
# Sol that doesn't exist yet
curl "${BASE_URL}/rovers/curiosity/photos?sol=999999&api_key=${API_KEY}"

# Returns empty array:
# { "photos": [] }
```

## Tips & Best Practices

1. **Cache Responses**: Manifests change slowly, cache them locally
2. **Use Earth Dates**: More intuitive than sol for most use cases
3. **Pagination**: Default is 25 photos/page, use `page` parameter for more
4. **Latest Photos**: Use `/latest_photos` endpoint for current mission activity
5. **Heroku Mirror**: Use for browser apps to avoid CORS issues
6. **Batch Requests**: Combine with jq/Python for bulk operations

## Troubleshooting

### CORS Issues in Browser

Use the Heroku endpoint:
```javascript
const BASE_URL = 'https://mars-photos.herokuapp.com/api/v1';
// No API key needed
```

### Rate Limit Exceeded

Get your own API key at https://api.nasa.gov/#signUp

### No Photos Found

Check:
1. Valid rover name
2. Sol/date within mission range
3. Camera is valid for that rover
4. Use manifest to find which sols have photos
