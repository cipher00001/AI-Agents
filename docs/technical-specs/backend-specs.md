# Backend Technical Specifications - Trip Planner AI

## 1. Introduction

This document outlines the technical specifications for the backend system of the Trip Planner AI application. The backend will be responsible for managing user data, trip information, interacting with AI agent(s) to generate itineraries, and providing APIs for the frontend client.

### Business Goals:
- Users can create trip itineraries.
- Provide suggestions for places to visit, things to do, and food.
- Offer route planning.
- Estimate trip costs.
- **Phase 2:** Personalized trip planning based on user preferences.

## 2. Technology Stack

-   **Programming Language**: Python 3.9+
-   **Web Framework**: **FastAPI**
    -   *Reasoning*: High performance, asynchronous support (beneficial for AI model interactions), automatic data validation, and OpenAPI/Swagger UI generation for clear API documentation.
-   **Database**:
    -   **Primary Choice**: **PostgreSQL** (via a managed service with a free tier like Supabase, Neon, ElephantSQL, or AWS RDS Free Tier).
        -   *Reasoning*: Robust, relational, well-supported, and suitable for structured trip data. Free tiers are available for initial development and low-traffic applications.
    -   **Alternative for specific use cases/simplicity**: **MongoDB Atlas** (Free Tier) if a NoSQL document model is preferred for highly flexible itinerary structures.
    -   **Development/Testing**: SQLite can be used for local development simplicity.
-   **AI Agent Integration**:
    -   The backend will communicate with the AI agent(s) likely via HTTP requests to an AI service/microservice endpoint. The AI agent is responsible for the core logic of itinerary generation, place suggestions, etc.
    -   The backend will send structured requests (e.g., JSON) to the AI agent and receive structured responses.
-   **Authentication**:
    -   **Method**: JWT (JSON Web Tokens)
    -   *Reasoning*: Stateless, widely adopted, and suitable for securing RESTful APIs.
-   **Deployment (Suggested)**:
    -   **Serverless Functions**: (e.g., Vercel for Python serverless functions, AWS Lambda, Google Cloud Functions)
    -   *Reasoning*: Scalability, potential cost-effectiveness (pay-per-use), and alignment with modern cloud architectures. Many serverless platforms offer generous free tiers.
-   **Task Queues (Optional, for longer AI processing)**:
    -   **Celery** with **Redis** or **RabbitMQ** as a broker.
    -   *Reasoning*: If AI itinerary generation takes significant time, offloading these tasks to a background worker queue will improve API responsiveness.

-   **External APIs (for contextual Trip Information - No-Cost/Minimal Cost Focus):**
    -   The backend may act as a proxy to these services to protect API keys and enable caching.
    -   **News APIs:**
        -   *Examples:* NewsAPI.org (free developer plan), GNews (free plan), Mediastack (free plan).
        -   *Use Case:* Fetch recent news relevant to the trip's destination.
        -   *Considerations:* Free tiers often have request limits, historical data restrictions, and may require attribution.
    -   **Hotel Listing/Pricing APIs:**
        -   *Challenge:* Comprehensive, real-time hotel listing and pricing APIs with generous free tiers are rare.
        -   *Potential Approach:* For a no-cost solution, this might rely on the AI generating suggestions for hotel *types* or known hotel names based on its training data, which the user would then research externally. The backend could have an extendable interface if a suitable freemium API is found later.
        -   *Placeholder:* If a specific API with a free tier is identified (e.g., some affiliate program APIs, though often with monetization requirements), it can be integrated.
    -   **Weather APIs:**
        -   *Examples:* OpenWeatherMap (free tier for current weather, forecast), WeatherAPI.com (free tier).
        -   *Use Case:* Fetch current weather and forecasts for the trip's destination and dates.

## 3. Data Models

```python
# Conceptual Pydantic models (FastAPI will use these for validation and serialization)

class User:
    id: int # Auto-incrementing primary key
    username: str # Unique
    email: str # Unique
    password_hash: str
    # For Phase 2:
    # preferences: Optional[Dict[str, Any]] = None # e.g., travel_style, interests
    created_at: datetime
    updated_at: datetime

class Trip:
    id: int # Auto-incrementing primary key
    user_id: int # Foreign key to User
    name: str
    destination_city: str
    destination_country: str
    start_date: date
    end_date: date
    budget: Optional[float] = None
    notes: Optional[str] = None
    # Phase 2:
    # travel_style: Optional[str] # e.g., 'adventure', 'relaxing', 'cultural'
    # interests: Optional[List[str]] # e.g., ['history', 'food', 'nature']
    created_at: datetime
    updated_at: datetime

class Place:
    id: int # Auto-incrementing primary key
    name: str
    description: Optional[str] = None
    place_type: str # e.g., 'attraction', 'restaurant', 'museum', 'park', 'viewpoint'
    address: Optional[str] = None
    city: str
    country: str
    latitude: Optional[float] = None
    longitude: Optional[float] = None
    estimated_duration_minutes: Optional[int] = None  # How long people typically spend here
    estimated_cost: Optional[float] = None  # Approximate entry fee or cost
    opening_hours: Optional[Dict[str, str]] = None  # e.g., {"monday": "9:00-17:00", ...}
    rating: Optional[float] = None  # Average rating if available
    image_url: Optional[str] = None  # URL to an image of the place
    external_url: Optional[str] = None  # Website or booking link
    source: str # 'ai_generated', 'user_added', 'osm' (OpenStreetMap), etc.
    metadata: Optional[Dict] = None  # Additional data like amenities, accessibility info
    created_at: datetime
    updated_at: datetime

class ShoppingVenue(Place):  # Inherits from Place but adds shopping-specific fields
    venue_type: str  # e.g., 'market', 'mall', 'boutique', 'artisan_shop', 'department_store', 'street_market'
    specialty_items: Optional[List[str]] = None  # e.g., ['antiques', 'local_crafts', 'fashion', 'food']
    price_level: str  # e.g., 'budget', 'mid-range', 'luxury'
    payment_methods: Optional[List[str]] = None  # e.g., ['cash', 'credit_card', 'mobile_payment']
    best_for: Optional[List[str]] = None  # e.g., ['souvenirs', 'local_products', 'luxury_goods']
    local_crafts: Optional[bool] = None  # Indicates if venue sells local/traditional crafts
    bargaining_common: Optional[bool] = None  # Whether bargaining/haggling is common here
    shipping_available: Optional[bool] = None  # Whether they offer international shipping
    tax_refund_available: Optional[bool] = None  # For tourist tax refund eligibility

class LocalProduct:
    id: int # Auto-incrementing primary key
    name: str
    description: str
    category: str  # e.g., 'crafts', 'food', 'fashion', 'art'
    typical_price_range: Optional[Dict[str, float]] = None  # e.g., {"min": 20, "max": 100}
    image_url: Optional[str] = None
    is_traditional: bool  # Whether it's a traditional/cultural item
    origin_region: Optional[str] = None  # Specific region/area known for this product
    materials: Optional[List[str]] = None  # e.g., ['silk', 'wood', 'ceramic']
    shipping_restrictions: Optional[List[str]] = None  # e.g., ['no_international', 'special_packaging_required']
    best_places_to_buy: Optional[List[Dict]] = None  # List of recommended shopping venues
    seasonal_availability: Optional[List[str]] = None  # e.g., ['summer', 'christmas_season']
    created_at: datetime
    updated_at: datetime

class Restaurant(Place):  # Inherits from Place but adds restaurant-specific fields
    cuisine_types: List[str]  # e.g., ['Italian', 'Pizza', 'Mediterranean']
    price_range: str  # e.g., '$', '$$', '$$$', '$$$$'
    dietary_options: Optional[List[str]] = None  # e.g., ['vegetarian', 'vegan', 'gluten-free']
    reservation_required: Optional[bool] = None
    delivery_available: Optional[bool] = None
    takeout_available: Optional[bool] = None
    popular_dishes: Optional[List[str]] = None
    meal_types: Optional[List[str]] = None  # e.g., ['breakfast', 'lunch', 'dinner']
    atmosphere: Optional[List[str]] = None  # e.g., ['casual', 'romantic', 'outdoor seating']

class LocalDish:
    id: int # Auto-incrementing primary key
    name: str
    description: str
    cuisine_type: str
    typical_price_range: Optional[Dict[str, float]] = None  # e.g., {"min": 10, "max": 20}
    image_url: Optional[str] = None
    is_vegetarian: Optional[bool] = None
    is_vegan: Optional[bool] = None
    spiciness_level: Optional[int] = None  # 0-4 scale
    best_time_to_try: Optional[List[str]] = None  # e.g., ['breakfast', 'dinner']
    seasonal_availability: Optional[List[str]] = None  # e.g., ['summer', 'fall']
    created_at: datetime
    updated_at: datetime

class TripFoodPreference:
    id: int # Auto-incrementing primary key
    trip_id: int # Foreign key to Trip
    preferred_cuisines: Optional[List[str]] = None
    dietary_restrictions: Optional[List[str]] = None
    price_range: Optional[List[str]] = None  # e.g., ['$', '$$']
    meal_preferences: Optional[Dict[str, str]] = None  # e.g., {"breakfast": "quick", "dinner": "fine_dining"}
    created_at: datetime
    updated_at: datetime

class TripPlace:
    id: int # Auto-incrementing primary key
    trip_id: int # Foreign key to Trip
    place_id: int # Foreign key to Place
    priority: Optional[int] = None  # User can prioritize places they definitely want to visit
    user_notes: Optional[str] = None  # User's personal notes about this place
    visited: bool = False  # Track if user has visited this place
    created_at: datetime
    updated_at: datetime

class ItineraryDay:
    id: int # Auto-incrementing primary key
    trip_id: int # Foreign key to Trip
    date: date
    day_notes: Optional[str] = None # Overall notes for the day
    # serialized_activities: List[ItineraryItem] # Populated on retrieval

class ItineraryItem:
    id: int # Auto-incrementing primary key
    day_id: int # Foreign key to ItineraryDay
    item_type: str # ENUM: 'activity', 'sightseeing', 'food', 'transit', 'accommodation', 'note'
    name: str
    description: Optional[str] = None
    start_time: Optional[time] = None
    end_time: Optional[time] = None
    location_name: Optional[str] = None
    location_address: Optional[str] = None
    location_lat: Optional[float] = None
    location_lon: Optional[float] = None
    estimated_cost: Optional[float] = None
    booking_details: Optional[str] = None # e.g., confirmation number, link
    source: str # 'user_added', 'ai_generated'
    # For transit:
    # transit_mode: Optional[str] # e.g., 'car', 'train', 'bus', 'walk'
    # transit_duration_minutes: Optional[int]
    # transit_distance_km: Optional[float]
    created_at: datetime
    updated_at: datetime

# Optional: For caching AI suggestions to reduce costs and latency
class AISuggestionCache:
    id: int
    query_hash: str # Hash of the input parameters to the AI
    suggestion_type: str # e.g., 'places', 'activities', 'food_itinerary'
    response_data: dict # The JSON response from the AI
    created_at: datetime
    expires_at: datetime

class TripShoppingPreference:
    id: int # Auto-incrementing primary key
    trip_id: int # Foreign key to Trip
    interested_categories: Optional[List[str]] = None  # e.g., ['local_crafts', 'fashion', 'food']
    budget_range: Optional[Dict[str, float]] = None  # e.g., {"souvenirs": 200, "fashion": 500}
    preferred_venue_types: Optional[List[str]] = None  # e.g., ['markets', 'boutiques']
    specific_items_wanted: Optional[List[str]] = None  # e.g., ['silk scarf', 'local tea']
    created_at: datetime
    updated_at: datetime
```

## 4. API Endpoints (RESTful with FastAPI)

Base URL: `/api/v1`

### 4.1. User Authentication (`/auth`)
-   `POST /auth/register`
    -   Request Body: `{"username": "user", "email": "user@example.com", "password": "securepassword"}`
    -   Response: `{"message": "User registered successfully"}` (or user details + access token)
-   `POST /auth/login`
    -   Request Body: `{"email_or_username": "user@example.com", "password": "securepassword"}`
    -   Response: `{"access_token": "jwt_token", "token_type": "bearer"}`
-   `GET /auth/me` (Protected)
    -   Headers: `Authorization: Bearer <access_token>`
    -   Response: User details (excluding password hash).

### 4.2. Trips (`/trips`)
-   `POST /trips` (Protected)
    -   Request Body: Trip data (e.g., `{"name": "Paris Adventure", "destination_city": "Paris", ...}`)
    -   Response: Created trip object.
-   `GET /trips` (Protected)
    -   Query Params: `?limit=10&offset=0`
    -   Response: List of user's trips.
-   `GET /trips/{trip_id}` (Protected)
    -   Response: Specific trip object with its itinerary (days and items).
-   `PUT /trips/{trip_id}` (Protected)
    -   Request Body: Updated trip data.
    -   Response: Updated trip object.
-   `DELETE /trips/{trip_id}` (Protected)
    -   Response: `204 No Content` or confirmation message.

### 4.3. Itinerary Management (within Trips)
- Itinerary items are typically managed as part of the `Trip` object or through dedicated sub-resource endpoints if granularity is needed (e.g., `POST /trips/{trip_id}/days/{day_id}/items`). For simplicity, initial design can manage itinerary items directly via `PUT /trips/{trip_id}` by updating the trip's itinerary structure.

### 4.4. AI Itinerary Generation (`/ai`)
-   `POST /ai/generate-full-itinerary` (Protected)
    -   Request Body: `{"trip_id": 1, "destination_city": "Rome", "destination_country": "Italy", "start_date": "YYYY-MM-DD", "end_date": "YYYY-MM-DD", "budget": 500, "preferences": {"travel_style": "cultural", "interests": ["history", "art"]}}` (Preferences optional for Phase 1)
    -   Response: The generated itinerary structure (list of days with items), which is then saved by the backend and associated with the `trip_id`. Or a task ID if using background processing.
        ```json
        {
          "trip_id": 1,
          "status": "processing_or_completed", // if async
          "itinerary": { /* Full itinerary structure */ }
        }
        ```
-   `GET /ai/suggestions/places` (Protected)
    -   Query Params: `?destination_city=Rome&interest=historical_sites&limit=5`
    -   Response: List of place suggestions.
-   `GET /ai/suggestions/activities` (Protected)
    -   Query Params: `?destination_city=Rome&category=outdoor&limit=5`
    -   Response: List of activity suggestions.
-   `GET /ai/suggestions/food` (Protected)
    -   Query Params: `?destination_city=Rome&cuisine=italian&limit=5`
    -   Response: List of food suggestions.

### 4.5. Route Planning (`/routes`) - (May proxy to an external service or AI)
-   `POST /routes/calculate` (Protected)
    -   Request Body: `{"waypoints": [{"lat": ..., "lon": ...}, {"lat": ..., "lon": ...}], "mode": "driving"}`
    -   Response: Route details (distance, duration, polyline/steps).

### 4.6. Costing (Derived or via AI)
-   The cost will primarily be an aggregation of `estimated_cost` from `ItineraryItem`s.
-   `GET /trips/{trip_id}/estimated-cost` (Protected)
    -   Response: `{"total_estimated_cost": 1250.75, "currency": "USD"}`

### 4.7. Contextual Trip Information (`/trips/{trip_id}/contextual-info`)
-   These endpoints would proxy requests to external services, allowing for API key management and caching by the backend.
-   `GET /trips/{trip_id}/contextual-info/news` (Protected)
    -   Query Params: `?destination_city=<city>&destination_country=<country_code_or_name>`
    -   Response: Structured news data (e.g., list of articles with title, snippet, source, URL, published_date).
-   `GET /trips/{trip_id}/contextual-info/weather` (Protected)
    -   Query Params: `?destination_city=<city>&destination_country=<country>&date=<YYYY-MM-DD>` (for a specific day or general forecast if date omitted)
    -   Response: Structured weather data (e.g., temperature, conditions, wind, humidity, forecast for several days).
-   `GET /trips/{trip_id}/contextual-info/hotels` (Protected) - *Subject to availability of a free/freemium API*
    -   Query Params: `?destination_city=<city>&destination_country=<country>&check_in_date=<YYYY-MM-DD>&check_out_date=<YYYY-MM-DD>&guests=<num>&budget_tier=<low/mid/high>`
    -   Response: Structured list of hotel suggestions (e.g., name, estimated price range, rating, amenities). If no API, this might return AI-generated textual suggestions.

### 4.8. Places Management (`/trips/{trip_id}/places`)

-   `GET /trips/{trip_id}/places` (Protected)
    -   Query Params: 
        -   `?type=<attraction/restaurant/etc>`
        -   `?visited=<true/false>`
        -   `?priority=<1,2,3>`
    -   Response: List of places associated with the trip.

-   `POST /trips/{trip_id}/places` (Protected)
    -   Request Body: Place details (name, description, type, etc.)
    -   Response: Created place object and its association with the trip.

-   `GET /trips/{trip_id}/places/{place_id}` (Protected)
    -   Response: Detailed information about a specific place.

-   `PUT /trips/{trip_id}/places/{place_id}` (Protected)
    -   Request Body: Updated place details
    -   Response: Updated place object.

-   `DELETE /trips/{trip_id}/places/{place_id}` (Protected)
    -   Response: Success confirmation.

-   `POST /trips/{trip_id}/places/suggest` (Protected)
    -   Request Body: 
        ```json
        {
            "preferences": {
                "types": ["museum", "park", "restaurant"],
                "interests": ["history", "nature"],
                "max_distance_km": 5,
                "budget_range": {"min": 0, "max": 100}
            }
        }
        ```
    -   Response: List of AI-suggested places based on trip context and preferences.

-   `POST /trips/{trip_id}/places/{place_id}/mark-visited` (Protected)
    -   Request Body: `{"visited": true, "notes": "Optional visit notes"}`
    -   Response: Updated TripPlace object.

### 4.9. Food and Dining Management (`/trips/{trip_id}/food`)

-   `GET /trips/{trip_id}/restaurants` (Protected)
    -   Query Params:
        -   `?cuisine_type=<type>`
        -   `?price_range=<$/$$/$$$/$$$$>`
        -   `?meal_type=<breakfast/lunch/dinner>`
        -   `?dietary_options=<vegetarian/vegan/etc>`
    -   Response: List of restaurants matching criteria.

-   `POST /trips/{trip_id}/food-preferences` (Protected)
    -   Request Body: Food preferences for the trip (cuisines, dietary restrictions, etc.)
    -   Response: Created/updated food preferences.

-   `GET /trips/{trip_id}/local-dishes` (Protected)
    -   Query Params: `?cuisine_type=<type>&dietary=<restriction>`
    -   Response: List of local dishes to try at the destination.

-   `POST /trips/{trip_id}/restaurants/suggest` (Protected)
    -   Request Body:
        ```json
        {
            "meal_type": "dinner",
            "cuisine_preferences": ["Italian", "Japanese"],
            "dietary_restrictions": ["vegetarian"],
            "price_range": ["$$", "$$$"],
            "atmosphere": ["romantic"],
            "location": {
                "lat": 123.456,
                "lon": 789.012,
                "max_distance_km": 2
            }
        }
        ```
    -   Response: AI-curated list of restaurant suggestions.

-   `GET /trips/{trip_id}/food-itinerary` (Protected)
    -   Query Params: `?date=<YYYY-MM-DD>`
    -   Response: Suggested meals/restaurants for the specified day based on:
        -   Trip's food preferences
        -   Location of other activities that day
        -   Local meal times/customs
        -   Restaurant opening hours and reservation requirements

### 4.10. Shopping and Local Products (`/trips/{trip_id}/shopping`)

-   `GET /trips/{trip_id}/shopping-venues` (Protected)
    -   Query Params:
        -   `?venue_type=<market/mall/boutique/etc>`
        -   `?specialty=<antiques/crafts/fashion/etc>`
        -   `?price_level=<budget/mid-range/luxury>`
        -   `?local_crafts=<true/false>`
    -   Response: List of shopping venues matching criteria.

-   `POST /trips/{trip_id}/shopping-preferences` (Protected)
    -   Request Body: Shopping preferences for the trip
    -   Response: Created/updated shopping preferences.

-   `GET /trips/{trip_id}/local-products` (Protected)
    -   Query Params:
        -   `?category=<crafts/food/fashion/etc>`
        -   `?is_traditional=<true/false>`
        -   `?price_range=<min-max>`
    -   Response: List of local products to look for at the destination.

-   `POST /trips/{trip_id}/shopping-venues/suggest` (Protected)
    -   Request Body:
        ```json
        {
            "interests": ["local_crafts", "fashion"],
            "budget_level": "mid-range",
            "preferred_venues": ["markets", "boutiques"],
            "specific_items": ["ceramics", "textiles"],
            "location": {
                "lat": 123.456,
                "lon": 789.012,
                "max_distance_km": 3
            }
        }
        ```
    -   Response: AI-curated list of shopping venue suggestions.

-   `GET /trips/{trip_id}/shopping-itinerary` (Protected)
    -   Query Params: `?date=<YYYY-MM-DD>`
    -   Response: Suggested shopping venues/activities for the specified day based on:
        -   Trip's shopping preferences
        -   Location of other activities that day
        -   Venue opening hours
        -   Local shopping customs/best times
        -   Special markets or events

## 5. AI Agent Interaction

-   **Overall Architecture**: The backend will serve as an orchestrator for one or potentially multiple specialized AI agents. This allows for a modular approach where different agents can handle specific tasks such as core itinerary logic, image generation for locations, meal suggestions, detailed route optimization, etc.

-   **Request Flow with Potential Multi-Agent System**:
    1.  Frontend sends a request to a backend API endpoint (e.g., `/ai/generate-full-itinerary` or `/ai/suggestions/place-image`).
    2.  Backend validates the request and gathers necessary context (trip details, user preferences, specific location for an image, etc.).
    3.  Backend formats a prompt or a structured request for the primary AI agent (e.g., an itinerary planning agent) or a specialized agent.
    4.  Backend sends the request to the relevant AI Agent's endpoint. This could involve:
        -   A single, powerful AI model capable of multiple tasks.
        -   A primary coordinating AI agent that delegates sub-tasks to other specialized AI agents.
        -   The backend itself calling a sequence of specialized AI agents (e.g., get itinerary -> for each location, call image generation agent -> call meal suggestion agent).
    5.  The AI Agent(s) process the input and generate their respective outputs.
        -   Specialized agents would focus on their domain: generating an image URL, suggesting a list of restaurants, providing route coordinates.
        -   If agents interact, the output of one (e.g., a list of suggested restaurants) might become input for another (e.g., an agent that fetches details or images for those restaurants).
    6.  The AI Agent(s) return structured responses (preferably JSON) to the backend (or to the orchestrating agent, which then consolidates and returns to the backend).
        -   Example for itinerary item: `{"name": "Colosseum Visit", "type": "sightseeing", "estimated_duration": "3 hours", "description": "Explore the ancient Roman amphitheater."}`
        -   Example for image generation: `{"place_name": "Colosseum", "image_url": "http://example.com/image.jpg"}`
        -   Example for meal suggestion: `{"location_name": "Trastevere", "meal_type": "dinner", "suggestions": [{"restaurant_name": "Da Enzo al 29", "cuisine": "Roman"}]}`
    7.  Backend receives the AI's response(s).
    8.  Backend parses the response(s), validates structure, and transforms it into the application's data models (e.g., updating `ItineraryItem` with an image URL or adding new suggested meal items).
    9.  Backend saves the AI-generated/enhanced data to the database.
    10. Backend returns a success response (or the consolidated/generated data) to the frontend.

-   **Error Handling**:
    -   Implement timeouts for AI agent requests.
    -   Handle API errors from the AI agent (e.g., rate limits, server errors).
    -   Validate the structure and content of the AI's response. If invalid, return an appropriate error or attempt a retry with modified parameters if applicable.
-   **Caching**:
    -   Implement caching for common AI requests (e.g., suggestions for popular destinations) to reduce latency and API call costs, using the `AISuggestionCache` model.

### 5.1. AI Agent Implementation (Suggested Technologies - Minimal/No Cost Focus)

Given the constraint of a no-cost or minimal-cost project, the technology choices for AI agents must prioritize open-source software and generous free tiers of cloud services. This approach requires accepting trade-offs in terms of performance, model capability, and increased development/maintenance effort.

-   **1. Open-Source Models (Self-Hosted or on Free Tiers):**
    -   **Text Generation/Reasoning (Itinerary, Suggestions, Q&A):**
        -   *Models:* Focus on smaller, efficient open-source Large Language Models (LLMs) that can run on local CPUs or modest free-tier cloud compute. Look for quantized versions (e.g., GGUF, AWQ, GPTQ) which require fewer resources.
            -   Examples: Smaller variants of Mistral (e.g., 7B if resources allow, or its quantized versions), Llama 2 (e.g., 7B, quantized), Phi-2, TinyLlama.
        -   *Tools for Running Locally/Easily:*
            -   **Ollama:** Simplifies downloading and running many popular open-source LLMs on local machines (macOS, Linux, Windows). Excellent for development and can serve models over a local API.
            -   **LM Studio:** Similar to Ollama, provides a GUI for discovering, downloading, and running local LLMs.
            -   **Hugging Face `transformers` library (Python):** For programmatic control, model loading, and inference if building custom agent logic. Requires more setup.
    -   **Image Generation:**
        -   *Models:* **Stable Diffusion** (and its variants).
        -   *Tools:*
            -   Libraries like Hugging Face `diffusers` (Python) for programmatic generation.
            -   User interfaces like Automatic1111 Stable Diffusion Web UI (requires local setup, can be resource-intensive).
            -   Consider highly optimized versions or smaller models if resource-constrained.
    -   **Embeddings (for semantic search, RAG - Retrieval Augmented Generation):**
        -   *Models:* **Sentence Transformers** (from Hugging Face) provide a wide range of open-source models for generating text embeddings (e.g., `all-MiniLM-L6-v2`).
        -   *Use:* Generate embeddings for places, descriptions, or user queries to enable semantic similarity searches.

-   **2. Orchestration Frameworks (Open Source):**
    -   **LangChain / LlamaIndex (Python):**
        -   *Use Case:* Essential for building sophisticated agent logic. They can chain calls to your (potentially locally-hosted) models, manage prompts, interact with vector stores, and orchestrate complex workflows.
        -   *Why:* Free, open-source, and highly flexible for working with various models and data sources.

-   **3. AI Agent Service Hosting & Compute (Free Tiers - The Main Challenge):**
    -   **API Framework for Agents:** **FastAPI (Python)** is recommended for building the API endpoints for your agents, aligning with the main backend stack.
    -   **Compute Options (Prioritize ease of use and most generous free tiers):**
        -   **Local Development Machine:** The default for initial development and testing at no cost.
        -   **PythonAnywhere:** Offers a free tier suitable for hosting Python web applications (like FastAPI agents). Good for simpler agents or those not requiring large model hosting.
        -   **Render / Railway / Fly.io:** These PaaS (Platform-as-a-Service) providers have free tiers that can host Docker containers or Python web apps. Resource limits (CPU, RAM, disk, network, uptime/sleep policies) are strict but might suffice for lightweight agents or very small, optimized models.
        -   **Hugging Face Spaces:** Provides a free tier for hosting models and interactive apps (Gradio, Streamlit). Can potentially serve a model via an API endpoint if it fits resource constraints.
        -   **Oracle Cloud Free Tier:** Often cited for more generous "Always Free" compute instances (VMs) and other services. Could potentially host smaller self-contained models if you are comfortable managing a VM.
        -   **Google Cloud Run / AWS Lambda / Azure Functions (Serverless Free Tiers):** Suitable for short-lived, event-driven agent tasks. Deploying custom models with large dependencies can be complex and might hit size or memory limits on free tiers.
    -   **Important Considerations for Free Compute:**
        -   **Resource Limits:** Free tiers are inherently limited. Large AI models (especially unquantized LLMs or image models) will be slow or impossible to run effectively.
        -   **Performance:** Expect higher latency and lower throughput.
        -   **"Cold Starts":** Serverless functions or apps on sleeping free tiers may have delays when first invoked.

-   **4. Databases (with Robust Free Tiers):**
    -   **Primary Relational Database & Vector Store:**
        -   **Supabase:** Excellent free tier for PostgreSQL. Crucially, it includes the `pgvector` extension, allowing PostgreSQL to function as a vector database for storing and querying embeddings (for RAG).
    -   **Alternative Relational Databases:**
        -   **Neon / ElephantSQL:** Offer well-regarded free tiers for PostgreSQL if Supabase isn't chosen.
    -   **NoSQL Document Database (if preferred):**
        -   **MongoDB Atlas:** Offers a generous M0 free cluster.
    -   **Local/Embedded Databases:**
        -   **SQLite:** Bundled with Python, perfect for local development or very simple, low-concurrency applications.
        -   **ChromaDB / FAISS (for vector storage):** Can be run locally or self-hosted within your application if you are managing embeddings directly.

-   **5. Specific Task AI - Open Source / Free Alternatives:**
    -   **Route Suggestion:**
        -   *Data:* **OpenStreetMap (OSM)** provides free global map data.
        -   *Routing Engines (Self-Hosted):*
            -   **OSRM (Open Source Routing Machine):** Fast, can be run in a Docker container.
            -   **Valhalla:** Another powerful open-source routing engine.
        -   *Python Libraries:* `OSMnx` to download OSM data for use with routing engines.
        -   *Hosting:* Deploy the chosen routing engine on your free-tier compute (e.g., a Docker container on Render/Fly.io or an Oracle Cloud VM).
    -   **Meal Suggestions:**
        -   This will likely rely heavily on the capabilities of your chosen self-hosted open-source LLM, combined with effective prompting.
        -   Optionally, curate a small knowledge base from publicly available recipe/restaurant data (respecting terms of service) and use the LLM or simpler NLP techniques to query against it.

-   **Recommended Minimal Cost Approach for Trip Planner:**
    1.  **Backend & Agent APIs:** FastAPI.
    2.  **LLM & Orchestration:** An open-source LLM (e.g., a quantized Mistral or Llama variant) run via Ollama (for local API access) and orchestrated by LangChain/LlamaIndex within your FastAPI application.
        -   *Enhancement:* AI agents can be designed to leverage data fetched from the contextual info endpoints (news, weather, potential hotel data) to provide more informed or dynamic suggestions (e.g., safety tips based on news, activity adjustments based on weather, hotel recommendations incorporating real-time (mocked/example) pricing if available).
    3.  **Image Generation:** Stable Diffusion (e.g., via `diffusers` in Python), called by an agent. Expect this to be slow on free compute.
    4.  **Database:** Supabase (PostgreSQL + `pgvector` free tier).
    5.  **Hosting:**
        -   Develop locally.
        -   Deploy the FastAPI app (including agent logic) to a PaaS free tier like Render or PythonAnywhere. If using Ollama for the LLM, it typically runs as a separate local server; for "deployment," you'd need to run the LLM on the same free-tier compute if possible (challenging for resource-heavy LLMs) or explore services that might allow hosting very small models. This is the hardest part for "no cost".
    6.  **Routing:** Self-hosted OSRM/Valhalla with OpenStreetMap data, containerized and deployed on a free PaaS tier if resources allow.

This "no-cost" approach prioritizes open-source tools and free service tiers. It requires significant developer effort for setup, optimization, and management, and an acceptance of limitations in performance and model capabilities.

## 6. Phase 2 Considerations

-   **Personalization Data**:
    -   Extend `User` model to store detailed preferences (travel style, activity preferences, food preferences, pace, accessibility needs).
    -   This data will be passed to the AI agent to tailor suggestions.
-   **Feedback Loop**:
    -   API endpoints for users to rate or provide feedback on AI-generated suggestions or itinerary items.
    -   Store this feedback to potentially fine-tune AI prompts or models in the future.
-   **Learning from Modifications**:
    -   Track user modifications to AI-generated itineraries. This data can be valuable for understanding user preferences and improving future AI outputs.

## 7. Non-Functional Requirements

-   **Scalability**:
    -   Utilize serverless architecture where possible.
    -   Design database schemas for efficient querying.
    -   Consider asynchronous task processing for long-running AI operations.
-   **Security**:
    -   Implement robust input validation (FastAPI helps).
    -   Secure JWT implementation (e.g., use strong secrets, appropriate algorithms, token expiry).
    -   Protect against common web vulnerabilities (OWASP Top 10).
    -   Rate limiting on sensitive endpoints.
-   **Maintainability**:
    -   Follow clean code principles.
    -   Modular design (e.g., separate modules for auth, trips, AI interaction).
    -   Comprehensive API documentation (auto-generated by FastAPI).
    -   Unit and integration tests.
-   **Cost-Effectiveness**:
    -   Leverage free tiers for database and deployment during development and early stages.
    -   Optimize AI agent calls (e.g., caching, batching requests if possible).
    -   Monitor resource usage.
-   **Reliability**:
    -   Proper error handling and logging.
    -   Health check endpoints.

## 8. Database Schema (Conceptual - PostgreSQL)

```sql
CREATE TABLE Users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(255) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    preferences JSONB, -- For Phase 2
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE Trips (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES Users(id) ON DELETE CASCADE,
    name VARCHAR(255) NOT NULL,
    destination_city VARCHAR(255) NOT NULL,
    destination_country VARCHAR(255),
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    budget NUMERIC(10, 2),
    notes TEXT,
    -- Phase 2 fields
    -- travel_style VARCHAR(100),
    -- interests TEXT[], -- Array of strings
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    CONSTRAINT dates_check CHECK (end_date >= start_date)
);

CREATE TABLE ItineraryDays (
    id SERIAL PRIMARY KEY,
    trip_id INTEGER NOT NULL REFERENCES Trips(id) ON DELETE CASCADE,
    date DATE NOT NULL,
    day_notes TEXT,
    UNIQUE(trip_id, date) -- A trip can only have one entry for a specific date
);

CREATE TYPE ITEM_TYPE AS ENUM ('activity', 'sightseeing', 'food', 'transit', 'accommodation', 'note');
CREATE TYPE ITEM_SOURCE AS ENUM ('user_added', 'ai_generated');

CREATE TABLE ItineraryItems (
    id SERIAL PRIMARY KEY,
    day_id INTEGER NOT NULL REFERENCES ItineraryDays(id) ON DELETE CASCADE,
    item_type ITEM_TYPE NOT NULL,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    start_time TIME,
    end_time TIME,
    location_name VARCHAR(255),
    location_address VARCHAR(500),
    location_lat NUMERIC(9, 6),
    location_lon NUMERIC(9, 6),
    estimated_cost NUMERIC(10, 2),
    booking_details TEXT,
    source ITEM_SOURCE NOT NULL DEFAULT 'user_added',
    -- Transit specific fields
    -- transit_mode VARCHAR(50),
    -- transit_duration_minutes INTEGER,
    -- transit_distance_km NUMERIC(8,2),
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE AISuggestionCache (
    id SERIAL PRIMARY KEY,
    query_hash VARCHAR(64) UNIQUE NOT NULL, -- SHA256 hash of query params
    suggestion_type VARCHAR(100) NOT NULL,
    response_data JSONB NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    expires_at TIMESTAMPTZ NOT NULL
);

CREATE TABLE Places (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    place_type VARCHAR(100) NOT NULL,
    address VARCHAR(500),
    city VARCHAR(255) NOT NULL,
    country VARCHAR(255) NOT NULL,
    latitude NUMERIC(9, 6),
    longitude NUMERIC(9, 6),
    estimated_duration_minutes INTEGER,
    estimated_cost NUMERIC(10, 2),
    opening_hours JSONB,
    rating NUMERIC(3, 2),
    image_url VARCHAR(255),
    external_url VARCHAR(255),
    source VARCHAR(100) NOT NULL,
    metadata JSONB,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE TripPlaces (
    id SERIAL PRIMARY KEY,
    trip_id INTEGER NOT NULL REFERENCES Trips(id) ON DELETE CASCADE,
    place_id INTEGER NOT NULL REFERENCES Places(id) ON DELETE CASCADE,
    priority INTEGER,
    user_notes TEXT,
    visited BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE Restaurants (
    id SERIAL PRIMARY KEY,
    place_id INTEGER NOT NULL REFERENCES Places(id) ON DELETE CASCADE,
    cuisine_types TEXT[],
    price_range VARCHAR(4),
    dietary_options TEXT[],
    reservation_required BOOLEAN,
    delivery_available BOOLEAN,
    takeout_available BOOLEAN,
    popular_dishes TEXT[],
    meal_types TEXT[],
    atmosphere TEXT[]
);

CREATE TABLE LocalDishes (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT NOT NULL,
    cuisine_type VARCHAR(100) NOT NULL,
    typical_price_range JSONB,
    image_url VARCHAR(255),
    is_vegetarian BOOLEAN,
    is_vegan BOOLEAN,
    spiciness_level INTEGER CHECK (spiciness_level BETWEEN 0 AND 4),
    best_time_to_try TEXT[],
    seasonal_availability TEXT[],
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE TripFoodPreferences (
    id SERIAL PRIMARY KEY,
    trip_id INTEGER NOT NULL REFERENCES Trips(id) ON DELETE CASCADE,
    preferred_cuisines TEXT[],
    dietary_restrictions TEXT[],
    price_range TEXT[],
    meal_preferences JSONB,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE ShoppingVenues (
    id SERIAL PRIMARY KEY,
    place_id INTEGER NOT NULL REFERENCES Places(id) ON DELETE CASCADE,
    venue_type VARCHAR(50) NOT NULL,
    specialty_items TEXT[],
    price_level VARCHAR(20) NOT NULL,
    payment_methods TEXT[],
    best_for TEXT[],
    local_crafts BOOLEAN,
    bargaining_common BOOLEAN,
    shipping_available BOOLEAN,
    tax_refund_available BOOLEAN
);

CREATE TABLE LocalProducts (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT NOT NULL,
    category VARCHAR(100) NOT NULL,
    typical_price_range JSONB,
    image_url VARCHAR(255),
    is_traditional BOOLEAN NOT NULL,
    origin_region VARCHAR(255),
    materials TEXT[],
    shipping_restrictions TEXT[],
    best_places_to_buy JSONB,
    seasonal_availability TEXT[],
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE TripShoppingPreferences (
    id SERIAL PRIMARY KEY,
    trip_id INTEGER NOT NULL REFERENCES Trips(id) ON DELETE CASCADE,
    interested_categories TEXT[],
    budget_range JSONB,
    preferred_venue_types TEXT[],
    specific_items_wanted TEXT[],
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_trips_user_id ON Trips(user_id);
CREATE INDEX idx_itinerarydays_trip_id ON ItineraryDays(trip_id);
CREATE INDEX idx_itineraryitems_day_id ON ItineraryItems(day_id);
CREATE INDEX idx_aisuggestioncache_expires_at ON AISuggestionCache(expires_at);
CREATE INDEX idx_places_name ON Places(name);
CREATE INDEX idx_tripplaces_trip_id ON TripPlaces(trip_id);
CREATE INDEX idx_tripplaces_place_id ON TripPlaces(place_id);
CREATE INDEX idx_restaurants_cuisine_types ON Restaurants USING gin(cuisine_types);
CREATE INDEX idx_restaurants_price_range ON Restaurants(price_range);
CREATE INDEX idx_localdishes_cuisine_type ON LocalDishes(cuisine_type);
CREATE INDEX idx_shoppingvenues_venue_type ON ShoppingVenues(venue_type);
CREATE INDEX idx_shoppingvenues_price_level ON ShoppingVenues(price_level);
CREATE INDEX idx_localproducts_category ON LocalProducts(category);
CREATE INDEX idx_localproducts_is_traditional ON LocalProducts(is_traditional);
```

This provides a comprehensive starting point for your backend development.
