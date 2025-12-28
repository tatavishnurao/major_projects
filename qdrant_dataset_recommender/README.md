# Dataset Recommender

A FastAPI-based dataset recommendation system using Qdrant vector database for similarity search.

## Features

- **Vector-based similarity search**: Find similar datasets using Qdrant vector database
- **Multiple data types**: Support for air quality, climate, and custom datasets
- **RESTful API**: FastAPI with automatic documentation (Swagger/OpenAPI)
- **Docker support**: Easy deployment with Docker Compose
- **Extensible architecture**: Modular design for adding new data types and features

## Project Structure

```
dataset-recommender/
├── .gitignore
├── .python-version        # Python 3.11.4
├── .env.example          # Example environment variables
├── .env                  # Local environment configuration
├── pyproject.toml        # Project dependencies and metadata
├── docker-compose.yml    # Docker services definition
├── Dockerfile           # API container image
├── README.md            # This file
│
├── src/
│   └── dataset_recommender/
│       ├── __init__.py
│       ├── main.py                 # FastAPI application
│       ├── config.py               # Configuration settings
│       ├── models.py               # Pydantic models
│       ├── api.py                  # API endpoints
│       ├── qdrant_client.py        # Qdrant operations wrapper
│       ├── vector_extractor.py     # Vector extraction utilities
│       └── recommender.py          # Core recommendation logic
│
├── scripts/
│   ├── setup_qdrant.py            # Initialize Qdrant collection
│   └── ingest_sample_data.py      # Download and process datasets
│
├── tests/
│   ├── test_vectors.py            # Vector extraction tests
│   └── test_recommendations.py    # Recommendation logic tests
│
└── data/
    └── sample_datasets/            # Sample data files
```

## Quick Start

### Prerequisites

- Python 3.11+
- Docker and Docker Compose (for containerized setup)
- Kaggle API credentials (for dataset download)

### Installation

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd dataset-recommender
   ```

2. **Set up Python environment**
   ```bash
   # Using uv (recommended)
   uv sync
   
   # Or using venv
   python -m venv venv
   source venv/bin/activate
   pip install -e .
   ```

3. **Configure environment**
   ```bash
   cp .env.example .env
   # Edit .env with your settings
   ```

4. **Set up Kaggle API** (optional, for downloading datasets)
   ```bash
   mkdir -p ~/.config/kaggle
   # Download kaggle.json from https://www.kaggle.com/settings/account
   # Place it in ~/.config/kaggle/
   chmod 600 ~/.config/kaggle/kaggle.json
   ```

### Running Locally

1. **Start Qdrant**
   ```bash
   docker run -p 6333:6333 -p 6334:6334 -v qdrant_storage:/qdrant/storage qdrant/qdrant
   ```

2. **Download and prepare datasets**
   ```bash
   uv run python scripts/ingest_sample_data.py
   ```

3. **Initialize Qdrant collection**
   ```bash
   uv run python scripts/setup_qdrant.py
   ```

4. **Start the API server**
   ```bash
   uv run uvicorn src.dataset_recommender.main:app --reload
   ```

   The API will be available at `http://localhost:8000`

### Running with Docker Compose

```bash
# Build and start all services
docker-compose up -d

# View logs
docker-compose logs -f api

# Stop services
docker-compose down
```

## API Documentation

Once the server is running, visit:
- **Swagger UI**: `http://localhost:8000/docs`
- **ReDoc**: `http://localhost:8000/redoc`
- **OpenAPI JSON**: `http://localhost:8000/openapi.json`

### Available Endpoints

#### Health Check
```bash
GET /health
```

#### Search by Vector
```bash
POST /search/vector
Content-Type: application/json

{
  "values": [1.0, 0.5, 0.3, 0.2, 0.1],
  "limit": 10,
  "threshold": 0.5
}
```

#### Search by Air Quality Parameters
```bash
GET /search/air-quality?pm25=35.5&pm10=50.2&no2=25.3&so2=15.1&co=0.8&limit=10
```

#### List Available Datasets
```bash
GET /datasets
```

#### Get Dataset Information
```bash
GET /datasets/{collection}
```

## Vector Format

### Air Quality Vector (5 dimensions)
1. **PM2.5**: Particulate matter < 2.5 micrometers (µg/m³)
2. **PM10**: Particulate matter < 10 micrometers (µg/m³)
3. **NO2**: Nitrogen dioxide (ppb)
4. **SO2**: Sulfur dioxide (ppb)
5. **CO**: Carbon monoxide (ppm)

All vectors are normalized to unit length for cosine similarity search.

## Configuration

Edit `.env` file to customize:

```env
# Qdrant Settings
QDRANT_HOST=localhost
QDRANT_PORT=6333
QDRANT_API_KEY=

# Application Settings
LOG_LEVEL=info
DATA_PATH=./data/sample_datasets
```

## Testing

Run tests with pytest:

```bash
# Run all tests
uv run pytest tests/

# Run with coverage
uv run pytest --cov=src tests/

# Run specific test file
uv run pytest tests/test_vectors.py
```

## Development

### Code Style
- Uses `black` for formatting
- `isort` for import organization
- `mypy` for type checking

```bash
# Format code
uv run black src tests

# Check imports
uv run isort src tests

# Type check
uv run mypy src
```

## Data Sources

Current datasets:
- **Air Quality**: India Air Quality Data from Kaggle
  - Dataset ID: `rohanrao/air-quality-data-in-india`
  - Features: PM2.5, PM10, NO2, SO2, CO measurements

To add more datasets:
1. Download the dataset
2. Convert to JSONL format using vector_extractor
3. Run setup_qdrant.py to ingest
4. Add collection configuration to config.py

## Architecture

### Components

1. **API Layer** (`main.py`, `api.py`): FastAPI endpoints and request handling
2. **Recommendation Engine** (`recommender.py`): Core similarity search logic
3. **Vector Management** (`vector_extractor.py`): Feature extraction and normalization
4. **Database Client** (`qdrant_client.py`): Qdrant operations wrapper
5. **Configuration** (`config.py`): Centralized settings management
6. **Models** (`models.py`): Pydantic request/response schemas

### Data Flow

```
Raw Dataset
    ↓
Vector Extraction (vector_extractor.py)
    ↓
Normalization (L2 norm)
    ↓
Qdrant Ingestion (setup_qdrant.py)
    ↓
Query Vector → Search (recommender.py)
    ↓
Ranked Results (cosine similarity)
```

## Future Enhancements

- [ ] Support for additional dataset types
- [ ] Advanced vector extraction methods (embeddings)
- [ ] User feedback and ranking optimization
- [ ] Batch query processing
- [ ] Caching and query optimization
- [ ] Dataset metadata enrichment

## Contributing

1. Create a feature branch
2. Make your changes
3. Write tests
4. Submit a pull request

## License

MIT

## Support

For issues and questions, please open an issue on GitHub.

