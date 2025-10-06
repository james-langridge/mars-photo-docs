# Mars Rover Photos API Documentation

Interactive documentation for NASA's Mars Rover Photos API - access photo data from Curiosity, Perseverance, Opportunity, and Spirit rovers.

## 📖 View Documentation

**Choose your preferred style:**

- **[Redoc](https://james-langridge.github.io/mars-photo-docs/)** - Clean, professional layout (best for reading)
- **[Swagger UI](https://james-langridge.github.io/mars-photo-docs/swagger.html)** - Interactive testing (try API calls in browser)
- **[RapiDoc](https://james-langridge.github.io/mars-photo-docs/rapidoc.html)** - Modern dark theme
- **[Compare All Three](https://james-langridge.github.io/mars-photo-docs/compare.html)** - Feature comparison

## 🚀 Quick Start

### Get an API Key

```bash
# Free API key from NASA
https://api.nasa.gov/#signUp

# Rate Limits:
# DEMO_KEY: 50 requests/hour
# Personal key: 1,000 requests/hour
```

### Example Request

```bash
# Get Curiosity photos from Sol 1000
curl "https://api.nasa.gov/mars-photos/api/v1/rovers/curiosity/photos?sol=1000&api_key=YOUR_API_KEY"
```

### Available Endpoints

- `GET /rovers` - List all rovers
- `GET /rovers/{rover}/photos` - Query photos by sol/date/camera
- `GET /rovers/{rover}/latest_photos` - Get latest photos
- `GET /manifests/{rover}` - Mission manifest with photo inventory
- `GET /photos/{id}` - Get specific photo

## 📚 More Examples

See [examples.md](examples.md) for comprehensive usage examples in curl, JavaScript, and Python.

## 🛠️ Technical Details

This documentation is generated from an OpenAPI 3.0.3 specification:

- **Spec**: [openapi.yaml](openapi.yaml)
- **Renderers**: Redoc, Swagger UI, RapiDoc
- **Hosting**: GitHub Pages

## 🔗 API Information

- **Official API**: https://api.nasa.gov/mars-photos/api/v1
- **API Maintainer**: Chris Cerami (chrisccerami@gmail.com)
- **API Source Code**: https://github.com/chrisccerami/mars-photo-api
- **NASA Portal**: https://api.nasa.gov/

## 📝 License

Documentation: MIT
Mars Rover Photos API: See [API License](https://github.com/chrisccerami/mars-photo-api/blob/master/License.md)
