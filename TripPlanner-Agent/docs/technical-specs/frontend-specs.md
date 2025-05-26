# Frontend Technical Specifications - Trip Planner AI

## 1. Introduction & Goals

This document outlines the technical specifications for the frontend of the Trip Planner AI application. The frontend will provide an intuitive and responsive user interface for users to plan trips, interact with AI-generated suggestions, and manage their itineraries.

**Primary Goals:**
-   Allow users to easily create, view, and modify trip plans.
-   Present AI-generated suggestions (places, activities, food, routes) in a clear and actionable manner.
-   Ensure a smooth and responsive user experience across common devices (desktop and mobile web).
-   Integrate seamlessly with the backend API.
-   Adhere to a no-cost/minimal-cost development and deployment strategy.

## 2. Technology Stack (No-Cost/Minimal Cost Focus)

-   **Core Framework/Library:**
    -   **React.js (with Vite or Create React App):**
        -   *Why:* Large community, extensive libraries, component-based architecture. Vite offers a fast development experience. Create React App is a well-established alternative.
    -   **Alternative: Vue.js (with Vite):**
        -   *Why:* Gentler learning curve for some, excellent performance, good tooling. Also a strong choice.
    -   **Alternative: Svelte/SvelteKit:**
        -   *Why:* Compiles to highly optimized vanilla JavaScript, potentially leading to very fast applications. Growing ecosystem.
    -   **Decision Point:** Choose one based on team familiarity or preference. React is often a safe bet due to its ecosystem.
-   **Styling:**
    -   **Tailwind CSS:**
        -   *Why:* Utility-first CSS framework that allows for rapid UI development without writing custom CSS for everything. Highly configurable and results in small production CSS bundles.
    -   **CSS Modules / Styled Components (if not Tailwind):**
        -   *Why:* Scoped CSS, good for component-based architectures.
    -   **Plain CSS/Sass:** Always an option, but might be slower for rapid development.
-   **State Management:**
    -   **React Context API (with `useReducer` for more complex state):**
        -   *Why:* Built into React, sufficient for many small to medium applications, avoids adding another dependency.
    -   **Zustand / Jotai (if more global/complex state is anticipated):**
        -   *Why:* Lightweight, simpler alternatives to Redux. Easier to pick up.
    -   **Avoid Redux initially** unless the application scales to a complexity that genuinely demands it, to keep things simpler and lighter.
-   **Routing:**
    -   **React Router** (for React)
    -   **Vue Router** (for Vue)
    -   **SvelteKit's built-in router** (for SvelteKit)
-   **API Communication:**
    -   **Fetch API (native browser API):**
        -   *Why:* Built-in, no extra library needed for simple GET/POST requests.
    -   **Axios (optional):**
        -   *Why:* Provides some conveniences like automatic JSON parsing, request/response interceptors, and better error handling, but adds a small dependency.
-   **Build Tool:**
    -   **Vite:** Recommended for React, Vue, and Svelte for its speed.
    -   **Create React App (uses Webpack internally)**
-   **Deployment (Free Tiers):**
    -   **Vercel:** Excellent free tier for static sites and frontend frameworks. Integrates well with GitHub for CI/CD. (Primary recommendation)
    -   **Netlify:** Similar to Vercel, strong free tier, good CI/CD.
    -   **GitHub Pages:** Free for static sites, can host React/Vue/Svelte builds.
    -   **Cloudflare Pages:** Generous free tier for static and Jamstack sites.

## 3. Key Features & Pages (Derived from UI and Backend Capabilities)

Based on the provided UI mockups and backend functionality:

1.  **Homepage / Landing Page (`/`)**
    -   Hero section: "Plan Your Dream Trip with AI"
    -   Brief overview of features (Smart Suggests, Easy Edits, Offline Access - though offline access might be a V2+ feature for a PWA).
    -   Testimonials/Social Proof.
    -   Call to Action (CTA): "Start Planning" / "Join for Free".
    -   Newsletter subscription.
    -   Footer with links (Product, Resources, Company, etc.).

2.  **Authentication Pages (`/login`, `/register`)**
    -   Simple forms for user login and registration.
    -   Links for password recovery (future feature).

3.  **Dashboard / My Trips Page (`/dashboard` or `/trips`)**
    -   List of user's existing trips (cards with trip name, destination, dates).
    -   CTA to create a new trip.

4.  **Create/Edit Trip Page (`/trips/new`, `/trips/{trip_id}/edit`)**
    -   Form fields for trip name, destination (city, country), start date, end date, budget (optional), notes.
    -   AI interaction trigger (e.g., "Generate Itinerary with AI" button after basic details are filled).

5.  **Itinerary View/Builder Page (`/trips/{trip_id}`) (or a dedicated Trip Dashboard/Details Page)**
    -   **Main View:** Displays the full itinerary, day by day.
    -   **Contextual Information Section (New):** Displayed prominently, possibly at the top or in a dedicated sidebar/tab system for the destination.
        -   **Latest News:**
            -   A feed of recent news headlines/snippets relevant to the destination (e.g., using a masonry layout for visual appeal if desired).
            -   Each news item could link to the original source.
            -   Fetched from `GET /trips/{trip_id}/contextual-info/news`.
        -   **Current Weather & Forecast:**
            -   Display current weather conditions (temperature, icon, description).
            -   Show a multi-day forecast (e.g., 3-5 days).
            -   Fetched from `GET /trips/{trip_id}/contextual-info/weather`.
        -   **Hotel Suggestions/Listings:**
            -   Display a list of suggested hotels (name, potential price range, rating if available).
            -   If a true API is used, could include images and booking links (though free tiers might not support deep linking for booking).
            -   If AI-generated, it might be a textual list of suggestions.
            -   Fetched from `GET /trips/{trip_id}/contextual-info/hotels`.
    -   **Places to Visit Section:**
        -   **Map View:**
            -   Interactive map showing all saved places for the trip
            -   Clustered markers for areas with multiple places
            -   Click/tap to view place details
            -   Option to view route between selected places
        -   **List View:**
            -   Grid or list of saved places with key information
            -   Filterable by type (attraction, restaurant, etc.)
            -   Sortable by priority, name, or distance
            -   Each place card shows:
                -   Image (if available)
                -   Name and type
                -   Brief description
                -   Estimated duration and cost
                -   Priority indicator
                -   Visited/Not visited status
        -   **Place Discovery:**
            -   Search bar for finding new places
            -   "Suggest Places" button to get AI recommendations
            -   Nearby places section
            -   Popular places in the destination
        -   **Place Management:**
            -   Add to trip button
            -   Set priority level
            -   Mark as visited/not visited
            -   Add personal notes
            -   Remove from trip
    -   **Itinerary Builder Section (as seen in UI):**
        -   List of itinerary items (e.g., "AM Yoga Session", "Sightseeing Tour") often with icons and times.
        -   Ability to add/edit/delete items manually.
        -   Drag-and-drop reordering of items within a day or between days (advanced feature, consider for V2).
    -   **Your Itinerary Section:** Displays the current state of the itinerary for a selected day.
    -   **Suggested Activities/Places/Food Section:**
        -   Cards displaying AI-generated suggestions (image, name, brief description).
        -   "Add to Itinerary" button for each suggestion.
        -   Possibly filterable (e.g., by type: activity, food, sightseeing).
    -   **Your Notes Section:** A simple text area for users to add personal notes to their trip or a specific day.
    -   **"Ask AI" Floating Button:** A prominent button to trigger further AI interactions, such as:
        -   "Suggest more activities for [Day X]"
        -   "Find restaurants near [Location Y]"
        -   "What's the best way to get from A to B?"

6.  **Destination Detail Page (as seen in "Wanderlust Explorer" UI - could be integrated or separate)**
    -   Destination Overview (image, description).
    -   Hotel Recommendations (cards with details, possibly links).
    -   Activities to Enjoy (cards with details).
    -   Accommodation, Food, Transport cost estimates (summary).
    -   This page seems more informational. Parts of it could be integrated into the itinerary suggestions or as a separate feature to explore destinations before creating a trip.

7.  **User Profile/Settings Page (`/profile` or `/settings`)**
    -   View/edit user details (username, email).
    -   Password change.
    -   Phase 2: Preferences for AI (travel style, interests).

8.  **Food & Dining Features**
    -   **Restaurant Discovery & Management:**
        -   Interactive map showing restaurants with filters for:
            -   Cuisine type
            -   Price range
            -   Dietary options
            -   Meal types (breakfast, lunch, dinner)
            -   Distance from accommodations/activities
        -   List view with detailed restaurant cards showing:
            -   Photos (if available)
            -   Basic info (name, cuisine, price range)
            -   Popular dishes
            -   Opening hours
            -   Dietary options
            -   Atmosphere tags
            -   Reservation status
        -   AI-powered restaurant suggestions based on:
            -   Trip preferences
            -   Daily itinerary
            -   Local specialties
            -   Group size and dietary restrictions
    
    -   **Local Cuisine Explorer:**
        -   Showcase of must-try local dishes with:
            -   Photos and descriptions
            -   Where to find them
            -   Price ranges
            -   Best times to try
            -   Dietary information
        -   Cultural context for local dining customs
        -   Seasonal specialties during the trip dates
    
    -   **Meal Planning Features:**
        -   Meal slot allocation in daily itinerary
        -   Restaurant reservation reminders
        -   Dietary restriction tracking
        -   Budget tracking for meals
        -   Integration with other activities (e.g., lunch near morning sightseeing spot)

9.  **Shopping & Local Products Features**
    -   **Shopping Venue Discovery & Management:**
        -   Interactive map showing shopping venues with filters for:
            -   Venue type (markets, malls, boutiques, etc.)
            -   Specialty items
            -   Price levels
            -   Local crafts availability
            -   Payment methods accepted
        -   List view with detailed venue cards showing:
            -   Photos (if available)
            -   Basic info (name, type, specialties)
            -   Price level indicators
            -   Opening hours
            -   Payment methods
            -   Special features (tax refund, shipping services)
            -   Local craft availability
        -   AI-powered shopping suggestions based on:
            -   Trip preferences
            -   Shopping interests
            -   Budget constraints
            -   Location and timing
    
    -   **Local Products Explorer:**
        -   Showcase of local specialties and traditional items:
            -   Product photos and descriptions
            -   Where to buy them
            -   Price ranges
            -   Materials and craftsmanship
            -   Shipping considerations
        -   Cultural context for shopping customs
        -   Seasonal products and special markets
    
    -   **Shopping Planning Features:**
        -   Shopping venue bookmarking
        -   Budget tracking for shopping
        -   Best times to visit different venues
        -   Integration with other activities
        -   Shopping district routes
        -   Bargaining tips where applicable
        -   Tax refund tracking

## 4. Component Breakdown (High-level)

-   `Layout` (Navbar, Footer, Main content area)
-   `Navbar` (Logo, Navigation links, User avatar/Login button, Help button)
-   `Footer`
-   `Button` (Primary, Secondary, Floating Action Button for "Ask AI")
-   `Card` (TripCard, SuggestionCard, ItineraryItemCard, RecommendationCard)
-   `Modal` (for forms, confirmations, AI interaction prompts)
-   `Forms` (AuthForm, TripForm, ItineraryItemForm)
-   `DatePicker`
-   `TripList` / `TripListItem`
-   `ItineraryDayView`
-   `ItineraryItem`
-   `SuggestionList` / `SuggestionItem`
-   `LoadingSpinner`
-   `Notification` / `Toast` (for API responses, errors)

-   `NewsFeed` / `NewsItem` (for displaying news articles)
-   `WeatherDisplay` (current weather and forecast)
-   `HotelList` / `HotelListItem` (for displaying hotel suggestions)

-   **Place-related Components:**
    -   `PlaceMap` (interactive map showing places)
    -   `PlaceList` / `PlaceCard`
    -   `PlaceDetails` (modal or side panel)
    -   `PlaceSearchBar`
    -   `PlaceFilters` (type, visited status, priority)
    -   `AddPlaceForm`
    -   `PlacePriorityPicker`
    -   `VisitedToggle`
    -   `NearbyPlaces`
    -   `PopularPlaces`

-   **Food & Dining Components:**
    -   `RestaurantMap`
    -   `RestaurantList` / `RestaurantCard`
    -   `RestaurantFilters`
    -   `CuisineSelector`
    -   `DietaryOptionsSelector`
    -   `PriceRangeSelector`
    -   `LocalDishList` / `LocalDishCard`
    -   `MealPlanner`
    -   `RestaurantSuggestionPanel`
    -   `DiningCustomsGuide`
    -   `MealTimingSelector`
    -   `ReservationStatus`
    -   `PopularDishesCarousel`

-   **Shopping Components:**
    -   `ShoppingVenueMap`
    -   `ShoppingVenueList` / `ShoppingVenueCard`
    -   `ShoppingVenueFilters`
    -   `VenueTypeSelector`
    -   `PriceLevelSelector`
    -   `LocalProductList` / `LocalProductCard`
    -   `ShoppingPlanner`
    -   `VenueSuggestionPanel`
    -   `ShoppingCustomsGuide`
    -   `BudgetTracker`
    -   `TaxRefundCalculator`
    -   `ShoppingRouteMap`
    -   `SeasonalMarketsCalendar`
    -   `BargainingTipsPanel`

## 5. State Management Strategy

-   **User Authentication:** Store JWT token (e.g., in `localStorage` or `sessionStorage`) and user information in a global context or Zustand/Jotai store.
-   **Trip Data:**
    -   List of trips on the dashboard.
    -   Currently selected/active trip details, including its full itinerary.
    -   This active trip data will be the most complex piece of state, likely managed in a dedicated context or slice of a global store.
-   **UI State:** Loading states, modal visibility, selected day in itinerary, etc. Can be managed by local component state or a global UI state slice.
-   **AI Suggestions:** Temporary state to hold suggestions fetched from the API before they are added to an itinerary.
-   **Contextual Trip Information:** State to hold news, weather, and hotel data fetched for the current trip. This could be part of the active trip data state or managed separately if it refreshes independently.
-   **Places State:**
    -   List of places saved for the trip
    -   Currently selected place
    -   Place search results
    -   Map view state (center, zoom, visible places)
    -   Filters and sorting preferences

## 6. API Integration

-   Frontend will communicate with the backend via REST API calls as defined in `Backend-flask/Technical-Specs.md`.
-   Authentication will use JWT Bearer tokens in Authorization headers.
-   Implement clear loading and error states for all API interactions.
-   Requests that trigger AI generation might be long-running. The UI should provide feedback:
    -   Initial request to backend.
    -   If backend uses a task queue: UI polls for status or backend uses WebSockets (WebSockets are V2+ for complexity with free tiers).
    -   Simpler approach: Backend handles the AI call synchronously if it's reasonably fast (e.g., a few seconds), or the UI shows a loading state for a longer period. For very long tasks, an asynchronous polling mechanism is better.

## 7. User Stories

**Authentication:**
-   As a new user, I want to register for an account so I can create and save my trips.
-   As an existing user, I want to log in to my account so I can access my saved trips.
-   As a logged-in user, I want to log out of my account to secure my information.

**Trip Management:**
-   As a user, I want to create a new trip by providing a name, destination, and dates so I can start planning.
-   As a user, I want to view a list of all my created trips so I can easily access them.
-   As a user, I want to edit the basic details of an existing trip (name, destination, dates, budget).
-   As a user, I want to delete a trip I no longer need.

**Itinerary Planning & AI Interaction:**
-   As a user, after creating a trip, I want to be able to request the AI to generate a full itinerary for my destination and dates.
-   As a user, I want to view the AI-generated itinerary, broken down by day.
-   As a user, I want to see AI-suggested activities, places to visit, and food options relevant to my trip destination.
-   As a user, I want to be able to add an AI-suggested item (activity, place, food) to a specific day in my itinerary.
-   As a user, I want to manually add a custom item (activity, note, meal) to a specific day in my itinerary.
-   As a user, I want to edit the details of an item in my itinerary (name, time, notes, cost).
-   As a user, I want to remove an item from my itinerary.
-   As a user, I want to be able to ask the AI for more specific suggestions (e.g., "more restaurants near X," "family-friendly activities for Day 2") via an "Ask AI" feature.
-   As a user, I want to add general notes to my trip or to a specific day.

**Contextual Information on Trip Details Page:**
-   As a user, when viewing my trip details, I want to see the latest news for my destination so I can be aware of current events.
-   As a user, when viewing my trip details, I want to see the current weather and a short-term forecast for my destination to help me plan activities and packing.
-   As a user, when viewing my trip details, I want to see hotel suggestions or listings for my destination to help me with accommodation planning.

**Information & Exploration (from "Wanderlust Explorer" UI concepts):**
-   As a user, I want to explore details about a destination, including an overview, hotel recommendations, and activities to enjoy, to get ideas before or during trip planning.
-   As a user, I want to see estimated costs for accommodation and food for a destination.

**User Profile (Phase 2):**
-   As a user, I want to update my profile information (e.g., email).
-   As a user, I want to change my password.
-   As a user, I want to set my travel preferences (style, interests) so the AI can provide more personalized suggestions.

**Places Management:**
-   As a user, I want to see all the places I can visit during my trip on an interactive map.
-   As a user, I want to search for specific places in my destination.
-   As a user, I want to get AI-suggested places based on my interests and trip context.
-   As a user, I want to save places I'm interested in visiting to my trip.
-   As a user, I want to set priority levels for places I definitely want to visit.
-   As a user, I want to mark places as visited once I've been there.
-   As a user, I want to add personal notes about places I'm interested in or have visited.
-   As a user, I want to see estimated visit duration and costs for places to help with planning.
-   As a user, I want to filter places by type (attraction, restaurant, etc.) to help organize my planning.
-   As a user, I want to see popular places in my destination to ensure I don't miss major attractions.
-   As a user, I want to find places near other places I'm visiting to optimize my itinerary.
-   As a user, I want to remove places from my trip if I decide not to visit them.
-   As a user, I want to see a route between selected places to understand travel times and logistics.

**Food & Dining:**
-   As a user, I want to set my dietary preferences and restrictions for the trip.
-   As a user, I want to discover local dishes that match my dietary requirements.
-   As a user, I want to see restaurant suggestions near my planned activities.
-   As a user, I want to view restaurants on a map to understand their location relative to my accommodations and activities.
-   As a user, I want to get AI-suggested restaurants based on my preferences and trip context.
-   As a user, I want to learn about local dining customs and etiquette.
-   As a user, I want to see which restaurants require reservations in advance.
-   As a user, I want to track my meal-related expenses within my trip budget.
-   As a user, I want to plan my meals around my daily activities.
-   As a user, I want to save restaurants to try during my trip.
-   As a user, I want to mark restaurants as visited and add my personal notes/ratings.
-   As a user, I want to see popular local dishes and where to find them.
-   As a user, I want to filter restaurants by cuisine type, price range, and dietary options.
-   As a user, I want to understand meal timing customs in my destination (e.g., when locals typically eat dinner).
-   As a user, I want to get suggestions for authentic local food experiences (e.g., food markets, cooking classes).

**Shopping & Local Products:**
-   As a user, I want to set my shopping preferences and interests for the trip.
-   As a user, I want to discover local products and traditional crafts.
-   As a user, I want to see shopping venues near my planned activities.
-   As a user, I want to view shopping venues on a map to understand their locations relative to my accommodations and activities.
-   As a user, I want to get AI-suggested shopping venues based on my interests and trip context.
-   As a user, I want to learn about local shopping customs and bargaining etiquette.
-   As a user, I want to know which venues offer tax refunds for tourists.
-   As a user, I want to track my shopping expenses within my trip budget.
-   As a user, I want to plan my shopping around my daily activities.
-   As a user, I want to save shopping venues to visit during my trip.
-   As a user, I want to mark venues as visited and add my personal notes/ratings.
-   As a user, I want to see popular local products and where to find them.
-   As a user, I want to filter shopping venues by type, price level, and specialties.
-   As a user, I want to understand shopping customs in my destination (e.g., bargaining, opening hours).
-   As a user, I want to get suggestions for authentic local shopping experiences (e.g., traditional markets, artisan workshops).
-   As a user, I want to know about seasonal markets or shopping events during my trip dates.
-   As a user, I want to understand shipping options for larger purchases.
-   As a user, I want to create a shopping checklist of items I want to buy.
-   As a user, I want to see recommended shopping routes or districts.

## 8. Non-Functional Requirements

-   **Responsiveness:** The application must be responsive and usable on common desktop, tablet, and mobile web browsers.
-   **Performance:** Aim for fast load times and smooth interactions. Optimize images and leverage browser caching. Code splitting by route.
-   **Accessibility (A11Y):** Follow basic web accessibility guidelines (e.g., semantic HTML, keyboard navigation, ARIA attributes where appropriate) to make the app usable by a wider audience.
-   **Usability:** Intuitive navigation and clear visual hierarchy.
-   **Maintainability:** Well-structured, commented (where non-obvious), and modular code.
-   **Cost:** All chosen technologies and deployment methods must align with the no-cost/minimal-cost constraint.

This provides a solid foundation for the frontend development. The UI mockups are a great guide, and the specific AI interactions will depend on the final API endpoints and capabilities exposed by the backend AI agents. 