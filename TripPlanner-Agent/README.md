# Trip Planner AI

An intelligent travel planning application that helps users create personalized trip itineraries with AI assistance.

## Features

- **Smart Trip Planning**: AI-powered itinerary generation based on user preferences
- **Local Insights**: Discover places, restaurants, and shopping venues with contextual information
- **Interactive Maps**: Visualize your trip with interactive maps and route planning
- **Food & Dining**: Find local cuisine, restaurants, and food experiences
- **Shopping Guide**: Discover local markets, products, and shopping districts
- **Cultural Context**: Learn about local customs, dining etiquette, and shopping practices
- **Budget Management**: Track expenses across activities, dining, and shopping

## Technology Stack

### Backend
- Python with FastAPI
- PostgreSQL Database
- AI Agents for intelligent suggestions
- JWT Authentication

### Frontend
- React.js with Vite
- Tailwind CSS
- Interactive Maps
- Responsive Design

## Project Structure

```
Trip-Planner-AI/
├── backend/
│   ├── app/
│   ├── tests/
│   ├── requirements.txt
│   └── README.md
├── frontend/
│   ├── src/
│   ├── public/
│   ├── package.json
│   └── README.md
└── docs/
    ├── technical-specs/
    ├── api-docs/
    └── ui-specs/
```

## Getting Started

### Prerequisites
- Python 3.9+
- Node.js 16+
- PostgreSQL

### Installation

1. Clone the repository:
```bash
git clone https://github.com/cipher00001/AI-Agents.git
cd AI-Agents/Trip-Planner-AI
```

2. Set up backend:
```bash
cd backend
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

3. Set up frontend:
```bash
cd frontend
npm install
```

### Running the Application

1. Start the backend server:
```bash
cd backend
uvicorn app.main:app --reload
```

2. Start the frontend development server:
```bash
cd frontend
npm run dev
```

## Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgments

- OpenStreetMap for map data
- Various open-source libraries and tools used in the project 