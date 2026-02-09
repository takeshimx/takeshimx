# 🌴 Travel Planner App

A comprehensive travel planning and expense management application built with modern web technologies. Manage your trips, track expenses, organize bookings, analyze travel costs with BigQuery integration, and enjoy a beautiful dark mode interface.

[日本語版 README](./README_JP.md)

## 🚀 Tech Stack

### Frontend
- **React 18** with TypeScript
- **Vite** for fast development and optimized builds
- **React Router v7** for navigation
- **Tailwind CSS** for responsive UI design with dark mode
- **Framer Motion** for smooth animations
- **React Dropzone** for file uploads

### Backend
- **FastAPI** (Python) for REST API
- **Firebase Firestore** for real-time database
- **Firebase Authentication** for user management
- **Google BigQuery** for analytics and cross-project expense data
- **Gmail API** for automatic booking confirmation extraction
- **Google Gemini 2.5 Flash** for AI analysis
- **Firecrawl SDK** for web scraping
- **Tavily Search API** for hotel search

## 🎯 Key Features

### 🗺️ Trip Management
- **Create & Organize Trips**: Set destinations, dates, and budgets
- **Multiple Destinations**: Track complex itineraries
- **Status Management**: Planning, Active, Completed, Cancelled
- **Budget Planning**: Set and monitor trip budgets
- **Google Sheets Import**: Bulk import trips from spreadsheets

### 💰 BigQuery Expense Integration (NEW)
- **Cross-Project Data Access**: Fetch expense data from separate Finance app BigQuery
- **Real-Time Expense Display**: View expenses from `project-2b-cbs.cbs_data.travel_expenses`
- **Expense Summary Dashboard**: 
  - Total expenses with category breakdown
  - Visual progress bars for spending categories
  - Date range filtering
- **Daily Food Expense Tracking**: Dedicated visualization for B&O -F&D expenses
  - Daily breakdown with bar charts
  - Total and average calculations
- **Detailed Expense Table**: 
  - Sortable by date (most recent first)
  - Multi-currency support (JPY, local currency)
  - Payment method tracking
  - Subcategory-based categorization
- **Dark Mode UI**: Sleek dark theme with vibrant gradients

### 📧 Automated Booking Management
- **Gmail Integration**: Automatically extract booking confirmations from emails
  - Flight bookings
  - Hotel reservations
  - Car rentals
  - Activity bookings
- **AI-Powered Parsing**: Automatic extraction of booking details
- **Review Before Approval**: Verify and edit auto-extracted bookings
- **Manual Entry**: Add bookings manually when needed
- **Timeline View**: Visual timeline of all bookings

### 🤖 AI-Powered Hotel Recommendation System
- **Agentic AI Implementation**: Autonomous decision-making without LangChain
- **Multi-Source Data Integration**: Combines Google Maps + Booking.com ratings
- **Fake Review Detection**: Statistical analysis and AI-powered trust scoring (4-level system)
- **Personalized Recommendations**: Analysis based on user preferences (quietness, cleanliness, location, breakfast, value)
- **AI Reasoning Visualization**: View agent's decision-making steps
- **Web Scraping**: Accurate Booking.com data via Firecrawl SDK
- **Real-Time Analysis**: Fast review analysis with Gemini 2.5 Flash

### 📊 Analytics & Insights
- **BigQuery Integration**: Historical expense analysis across projects
- **Spending Patterns**: Visualize expenses by category, date, and trip
- **Budget vs Actual**: Compare planned budgets with actual spending
- **Category Breakdown**: Detailed subcategory analysis
- **Trend Visualization**: Interactive charts and graphs

### 🎨 Modern UI/UX
- **Dark Mode Design**: Beautiful dark theme throughout the app
- **Responsive Layout**: Mobile-first design with Tailwind CSS
- **Smooth Animations**: Framer Motion for delightful interactions
- **Collapsible Sections**: Expandable category breakdowns
- **English Interface**: Full English language support

### 🔐 Security & Authentication
- **Firebase Authentication**: Secure user authentication
- **Email Whitelist**: Email-based access control
- **Data Privacy**: User-specific data isolation
- **Secure APIs**: Protected endpoints with authentication middleware
- **Cross-Project IAM**: Proper BigQuery permissions setup

## 🏗️ Architecture

```
Frontend (React + Vite)  Backend (FastAPI)      Database
┌─────────────────┐          ┌──────────────────┐       ┌─────────────────────┐
│ - React/TypeScript│         │ - REST API       │       │ Firestore           │
│ - Vite Build    │          │ - Gmail API      │       │ - Trip Data         │
│ - Tailwind CSS  │          │ - AI Analysis    │       │ - Expense Data      │
│ - Dark Mode     │          │ - BigQuery API   │       │ - Booking Data      │
└─────────────────┘          └──────────────────┘       └─────────────────────┘
        │                            │                            │
        └────────────────────────────┼────────────────────────────┘
                                     │
                         ┌──────────────────────┐
                         │ Google Cloud         │
                         │ - Firebase Auth      │
                         │ - BigQuery (2 projects)│
                         │   * Travel App       │
                         │   * Finance App      │
                         │ - Gmail API          │
                         │ - Gemini AI          │
                         └──────────────────────┘
```

## 📁 Project Structure

```
travel-planner-app/
├── frontend/                    # React Frontend
│   ├── src/
│   │   ├── components/         # Reusable components
│   │   │   ├── common/        # Buttons, cards, modals
│   │   │   ├── trip/          # Trip-related components
│   │   │   ├── expense/       # BigQuery expense components
│   │   │   │   ├── BQExpenseSummaryCard.tsx
│   │   │   │   ├── BQExpenseTable.tsx
│   │   │   │   └── DailyFoodExpensesCard.tsx
│   │   │   ├── timeline/      # Timeline view
│   │   │   ├── booking/       # Booking management
│   │   │   └── hotel/         # AI hotel recommendation
│   │   ├── pages/             # Page components
│   │   │   ├── Dashboard.tsx
│   │   │   ├── TripExpensesPage.tsx
│   │   │   └── BookingManager.tsx
│   │   ├── hooks/             # Custom React hooks
│   │   │   └── useBQExpenses.ts
│   │   ├── services/          # API clients
│   │   │   ├── api.ts
│   │   │   └── expenseService.ts
│   │   ├── types/             # TypeScript type definitions
│   │   │   └── expense.ts
│   │   └── utils/             # Utility functions
│   └── package.json
│
├── backend/                     # FastAPI Backend
│   ├── app/
│   │   ├── main.py            # FastAPI application
│   │   ├── core/
│   │   │   └── config.py      # BigQuery config
│   │   ├── schemas/           # Pydantic models
│   │   │   └── expense.py     # BigQuery expense schemas
│   │   ├── api/
│   │   │   └── endpoints/
│   │   │       └── expenses_endpoints.py
│   │   ├── services/          # Business logic
│   │   │   ├── firebase.py   # Firestore operations
│   │   │   ├── gmail.py      # Gmail API integration
│   │   │   ├── bigquery_service.py
│   │   │   ├── expense_service.py  # BigQuery expense queries
│   │   │   ├── hotel_agent.py      # Agentic AI core
│   │   │   ├── gemini_service.py   # AI analysis
│   │   │   ├── firecrawl_service.py # Web scraping
│   │   │   └── fake_review_detector.py
│   │   └── utils/             # Helper functions
│   ├── requirements.txt
│   └── Dockerfile
│
└── PROJECT_PLAN.md             # Complete implementation guide
```

## 🛠️ Installation & Setup

### Prerequisites
- Node.js 18+
- Python 3.9+
- Firebase project with Firestore and Authentication enabled
- Google Cloud project with BigQuery and Gmail API enabled
- **Two GCP Projects** (recommended):
  - Travel Planner App project
  - Finance App project (for expense data)
- API Keys: Google Maps, Gemini, Tavily, Firecrawl

### Frontend Setup

```bash
cd frontend
npm install
npm run dev
```

Frontend will be available at `http://localhost:5173`

### Backend Setup

```bash
cd backend

# Create virtual environment
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Set up environment variables
cp .env.example .env
# Edit .env file with your credentials

# Run development server
uvicorn app.main:app --reload --port 8000
```

Backend API will be available at `http://localhost:8000`

### BigQuery Setup

1. **Finance App Project** (`project-2b-cbs`):
   - Create dataset: `cbs_data`
   - Create table: `travel_expenses` with schema:
     ```
     travel_expense_id, transaction_id, travel_id, travel_title,
     expense_date, item_name, item_category, item_subcategory,
     price_jpy, local_currency, local_price, payment_method,
     payment_provider, notes, created_at, updated_at
     ```

2. **Travel App Project**:
   - Grant service account permissions on Finance App:
     - `BigQuery Data Viewer`
     - `BigQuery Job User`

### Gmail OAuth Setup

1. **Create Web OAuth Client** in Google Cloud Console:
   - Go to APIs & Services → Credentials
   - Click "Create Credentials" → "OAuth 2.0 Client ID"
   - Application type: **Web application**
   - Name: `Travel Planner Web Client`
   - Authorized JavaScript origins:
     ```
     http://localhost:5173
     https://your-frontend-url.run.app
     ```
   - Authorized redirect URIs:
     ```
     http://localhost:5173
     https://your-frontend-url.run.app
     ```
   - Download the credentials JSON

2. **Store Credentials in Secret Manager**:
   ```bash
   # Upload web OAuth credentials to Secret Manager
   gcloud secrets create GOOGLE_OAUTH_CREDENTIALS \
     --data-file=web_client_credentials.json \
     --project=your-project-id
   
   # Grant Cloud Run service account access
   gcloud secrets add-iam-policy-binding GOOGLE_OAUTH_CREDENTIALS \
     --member="serviceAccount:your-service-account@your-project.iam.gserviceaccount.com" \
     --role="roles/secretmanager.secretAccessor" \
     --project=your-project-id
   ```

3. **Enable Gmail API**:
   - Go to APIs & Services → Library
   - Search for "Gmail API" and enable it

4. **User Authentication Flow**:
   - Users click "📧 Gmailと連携" button in the app
   - OAuth consent screen appears
   - After authorization, tokens are stored in Firestore `gmail_tokens` collection
   - Users can then sync bookings from Gmail

### Environment Variables

#### Frontend (.env)
```env
VITE_API_URL=http://localhost:8000
VITE_FIREBASE_API_KEY=your_firebase_api_key
VITE_FIREBASE_AUTH_DOMAIN=your_project.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=your_project_id
```

#### Backend (.env)
```env
# Travel App Project
GOOGLE_CLOUD_PROJECT=your_travel_app_project_id
FIREBASE_CREDENTIALS=./serviceAccountKey.json

# Finance App Project (for BigQuery expenses)
FINANCE_PROJECT_ID=project-2b-cbs
BIGQUERY_DATASET=cbs_data
BIGQUERY_TABLE=travel_expenses

# APIs
GMAIL_CREDENTIALS=./credentials.json
GEMINI_API_KEY=your_gemini_api_key
FIRECRAWL_API_KEY=your_firecrawl_api_key
TAVILY_API_KEY=your_tavily_api_key
GOOGLE_MAPS_API_KEY=your_google_maps_api_key
```

## 📱 Usage

1. **Sign In**: Authenticate with your Google account
2. **Create a Trip**: Set destination, dates, and budget
3. **View Expenses**: Click "Expenses" button on any trip card
   - See expense summary with category breakdown
   - View daily food expenses (B&O -F&D)
   - Browse detailed expense table
4. **Import Bookings**: Forward booking confirmation emails for automatic extraction
5. **Search Hotels**: Use AI-powered hotel recommendations
6. **Review & Approve**: Check auto-extracted bookings in pending section
7. **Analyze**: View spending patterns and budget adherence

## 🔧 Technical Highlights

### Frontend Engineering
- **Type Safety**: Full TypeScript implementation with strict typing
- **Modern Tooling**: Vite for ultra-fast development
- **Responsive Design**: Mobile-first approach with Tailwind CSS
- **Dark Mode**: Consistent dark theme with vibrant gradients
- **Performance**: Optimized builds and lazy loading
- **Animations**: Smooth transitions with Framer Motion
- **Error Handling**: Proper handling of empty responses (204 No Content)

### Backend Engineering
- **RESTful API**: Structured endpoints with proper HTTP methods
- **Route Prioritization**: Specific routes before generic parameter routes
- **Data Validation**: Pydantic models for request/response validation
- **Cross-Project BigQuery**: Querying data from separate GCP projects
- **Gmail Integration**: OAuth2 authentication and message parsing
- **AI Analysis**: Structured data extraction from unstructured emails
- **Agentic AI**: Autonomous hotel analysis without LangChain
- **Error Handling**: Comprehensive error handling and logging

### Database Design
- **Firestore Collections**:
  - `trips`: Trip metadata and planning
  - `expenses`: Real-time expense tracking (deprecated, using BigQuery)
  - `bookings`: Booking confirmations and details
  - `users`: User profiles and settings
- **BigQuery Tables** (Finance App):
  - `travel_expenses`: Detailed expense records with subcategories
  - Linked to trips via `travel_id`
  - Multi-currency support
  - Payment method tracking

## 🎯 Demonstrated Skills

- **Full-Stack Development**: End-to-end application development
- **Modern Web Technologies**: React, TypeScript, Vite, Tailwind CSS
- **Backend Development**: Python, FastAPI, RESTful API design
- **Cloud Services**: Firebase, Google Cloud Platform, BigQuery
- **Cross-Project Integration**: Multi-project GCP architecture
- **Email Integration**: Gmail API, OAuth2, automated parsing
- **AI Integration**: Structured data extraction from text
- **Agentic AI Implementation**: Building autonomous AI agents without LangChain
- **Multi-Source Data Integration**: Combining multiple APIs with reliability validation
- **Web Scraping**: Dynamic content extraction with Firecrawl SDK
- **Statistical Analysis**: Fake review detection with NumPy/SciPy
- **Database Design**: NoSQL (Firestore) and SQL (BigQuery) modeling
- **API Design**: RESTful architecture and documentation
- **UI/UX Design**: Dark mode, responsive design, smooth animations
- **DevOps**: Containerization with Docker

## 📊 Development Status

See [PROJECT_PLAN.md](./PROJECT_PLAN.md) for detailed implementation roadmap and progress.

### Current Status
- ✅ Frontend foundation (React + Vite + Tailwind)
- ✅ Backend foundation (FastAPI + Firestore)
- ✅ Gmail integration and booking extraction
- ✅ Trip management
- ✅ **BigQuery Expense Integration** (Cross-project data access)
- ✅ **Dark Mode UI** (Consistent theme across all pages)
- ✅ **English Language Support** (Full internationalization)
- ✅ AI-Powered Hotel Recommendation System (Agentic AI implementation)
- 🚧 Analytics dashboard
- 📋 Deployment pipeline

## 🚀 Deployment

### Frontend (Firebase Hosting)
```bash
cd frontend
npm run build
firebase deploy --only hosting
```

### Backend (Cloud Run)
```bash
cd backend
gcloud run deploy travel-planner-api \
  --source . \
  --region asia-northeast1 \
  --allow-unauthenticated
```

## 📝 License

This is a personal project developed to demonstrate full-stack development capabilities, cloud integration, and modern web development best practices.

---

**Note**: This project showcases modern web development practices including React 18, TypeScript, FastAPI, Firebase, cross-project BigQuery integration, Gmail API integration, Agentic AI implementation, and beautiful dark mode UI design.
