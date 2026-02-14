# Design Document

## System Architecture

### High-Level Architecture
```
┌─────────────────────────────────────────────────────────┐
│              User Interface Layer (Streamlit)           │
│  ┌──────────────────────────────────────────────────┐   │
│  │  Header: Title + Compact Import Box              │   │
│  └──────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────┐   │
│  │  Tab Navigation: Dashboard | Decisions | Copilot │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
┌───────▼────────┐  ┌──────▼──────┐  ┌────────▼────────┐
│   Dashboard    │  │ AI Decision │  │   AI Copilot    │
│   Analytics    │  │ Intelligence│  │                 │
│   - KPIs       │  │ - Search    │  │ - Q&A           │
│   - Charts     │  │ - Styled DF │  │ - Context       │
└───────┬────────┘  └──────┬──────┘  └────────┬────────┘
        │                   │                   │
        └───────────────────┼───────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
┌───────▼────────┐  ┌──────▼──────┐  ┌────────▼────────┐
│ Data Processing│  │  Enhanced   │  │   AI Services   │
│   (Pandas)     │  │  Decision   │  │  (Groq API)     │
│ - Groupby      │  │   Engine    │  │ - Insights      │
│ - Rolling Avg  │  │ - Gap Ratio │  │ - Copilot       │
└────────────────┘  └─────────────┘  └─────────────────┘
```

## Component Design

### 1. Main Application (app.py)

#### Purpose
Central orchestrator with enhanced UI/UX and improved decision logic.

#### Key Enhancements
- Compact header with inline import box
- Responsive 2-column layouts
- Toggleable data views
- Searchable decision table
- Dynamic priority scoring
- Dark-themed visualizations

#### Layout Structure
```python
Header (3:1 column ratio)
├── Left: Title + Caption
└── Right: Compact Import Box (styled HTML)

Tabs
├── Dashboard
│   ├── KPI Row (3 columns)
│   ├── Toggle: Raw Data Table
│   ├── Row 1: Demand vs Stock (2:1) + Donut Chart
│   └── Row 2: Sales Trend + Price Comparison
│
├── AI Decisions
│   ├── Search Bar
│   └── Styled DataFrame (color-coded risks)
│
└── AI Copilot
    ├── Text Input
    └── Response Display
```

#### Data Flow
1. User uploads CSV → Validation → Session storage
2. Demand prediction via groupby + rolling average
3. Route to tabs with persistent session state
4. Generate decisions/insights on-demand

### 2. Enhanced Decision Engine

#### Algorithm (Updated)
```python
FOR each product:
    stock = Current_Stock
    demand = Predicted_Demand
    gap = stock - demand
    gap_ratio = |gap| / (demand + 1)
    
    IF stock < demand × 0.7:
        decision = RESTOCK
        risk = CRITICAL
        base_score = 95
        score = min(base_score + gap_ratio × 5, 100)
    
    ELSE IF stock < demand:
        decision = RESTOCK
        risk = HIGH
        base_score = 85
        score = min(base_score + gap_ratio × 10, 100)
    
    ELSE IF stock > demand × 2:
        decision = DISCOUNT
        risk = HIGH
        base_score = 80
        score = min(base_score + gap_ratio × 10, 100)
    
    ELSE:
        decision = MAINTAIN
        risk = LOW
        base_score = 60
        score = min(base_score + gap_ratio × 10, 100)
```

#### Decision Matrix (Enhanced)

| Condition | Decision | Risk Level | Base Score | Score Calculation | Business Impact |
|-----------|----------|------------|------------|-------------------|-----------------|
| Stock < Demand × 0.7 | RESTOCK | CRITICAL | 95 | 95 + (gap_ratio × 5) | High revenue loss risk |
| Stock < Demand | RESTOCK | HIGH | 85 | 85 + (gap_ratio × 10) | Potential missed sales |
| Stock > Demand × 2 | DISCOUNT | HIGH | 80 | 80 + (gap_ratio × 10) | Inventory holding cost |
| Stock ≈ Demand | MAINTAIN | LOW | 60 | 60 + (gap_ratio × 10) | Stable inventory |

#### Key Improvements
- Dynamic scoring based on gap severity
- Four-tier risk classification
- More granular understock detection
- Gap ratio for priority weighting

### 3. AI Insights Module (ai_insights.py)

#### Purpose
Generate business intelligence using LLM.

#### Function: `generate_insights(data_summary)`
- **Input**: String with aggregated metrics
- **Output**: AI-generated insights
- **Model**: llama-3.1-8b-instant
- **Temperature**: 0.5 (focused responses)
- **Max Tokens**: 400

#### Prompt Structure
```
Role: AI retail analytics expert
Input: Data summary
Output:
  1. Business health score (0-100)
  2. Key risks
  3. Opportunities
  4. Recommended actions
```

### 4. AI Copilot Module (ai_copilot.py)

#### Purpose
Interactive Q&A for retail data queries.

#### Function: `ask_ai(question, context)`
- **Input**: User question + first 100 rows
- **Output**: Natural language recommendations
- **Model**: llama-3.1-8b-instant
- **Temperature**: 0.7 (conversational)
- **Max Tokens**: 500

#### Design Considerations
- Context limited to 100 rows for token efficiency
- No code in responses (business-focused)
- Conversational tone

### 5. AI Decision Model (ai_decision_model.py)

#### Purpose
ML model for decision classification (currently unused in main app).

#### Function: `train_ai_model()`
- **Features**: Demand_Gap, Price_Gap
- **Labels**: Discount, Restock, Reduce Price, Maintain
- **Algorithm**: Random Forest Classifier

#### Feature Engineering
```python
Demand_Gap = Current_Stock - Units_Sold
Price_Gap = Price - Competitor_Price
```

## Data Model

### Input Schema (CSV)
```
Product: string (required)
Category: string (required)
Units_Sold: integer (required)
Current_Stock: integer (required)
Price: float (required)
Competitor_Price: float (required)
Date: datetime (optional)
```

### Derived Fields
```python
Predicted_Demand = groupby("Product")["Units_Sold"]
                   .transform(lambda x: x.rolling(7, min_periods=1).mean())

gap = Current_Stock - Predicted_Demand
gap_ratio = abs(gap) / (Predicted_Demand + 1)
```

### Decision Output Schema (DataFrame)
```python
{
    "Product": string,
    "Decision": enum(DISCOUNT, RESTOCK, MAINTAIN),
    "Reason": string,
    "Risk": enum(HIGH, CRITICAL, LOW),
    "Impact": string,
    "Priority Score": integer (0-100)
}
```

## User Interface Design

### Header Design
```
┌─────────────────────────────────────────────────────────┐
│  🧠 AI Retail Decision Intelligence Platform            │
│  Intelligent retail analytics powered by AI.            │
│  Upload your dataset to generate insights...            │
│                                                          │
│                                    ┌──────────────────┐ │
│                                    │ 📂 Import Dataset│ │
│                                    │ [File Uploader]  │ │
│                                    └──────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

### Tab 1: Dashboard (Enhanced)

#### KPI Section
```
┌──────────────┬──────────────┬──────────────┐
│ Total Rows   │ Unique       │ Categories   │
│ 1,234        │ Products: 45 │ 8            │
└──────────────┴──────────────┴──────────────┘
```

#### Visualization Layout
```
Row 1:
┌─────────────────────────┬──────────────┐
│ Demand vs Stock (Bar)   │ Category     │
│                         │ Distribution │
│                         │ (Donut)      │
└─────────────────────────┴──────────────┘

Row 2:
┌─────────────────────────┬──────────────┐
│ Daily Sales Trend       │ Price vs     │
│ (Line Chart)            │ Market Price │
│                         │ (Line Chart) │
└─────────────────────────┴──────────────┘
```

### Tab 2: AI Decisions (Enhanced)

#### Search Interface
```
┌─────────────────────────────────────────┐
│ 🔍 Search Product: [____________]       │
└─────────────────────────────────────────┘
```

#### Styled Decision Table
```
┌─────────┬──────────┬────────┬──────────┬─────────┬──────┐
│ Product │ Decision │ Reason │ Risk     │ Impact  │ Score│
├─────────┼──────────┼────────┼──────────┼─────────┼──────┤
│ Item A  │ RESTOCK  │ ...    │ CRITICAL │ ...     │ 98   │
│         │          │        │ (RED)    │         │      │
├─────────┼──────────┼────────┼──────────┼─────────┼──────┤
│ Item B  │ DISCOUNT │ ...    │ HIGH     │ ...     │ 85   │
│         │          │        │ (ORANGE) │         │      │
└─────────┴──────────┴────────┴──────────┴─────────┴──────┘
```

### Tab 3: AI Copilot
```
┌─────────────────────────────────────────┐
│ Ask AI about your store data:           │
│ [_________________________________]     │
│                                         │
│ Response:                               │
│ Based on your data...                   │
└─────────────────────────────────────────┘
```

## Visualization Design

### Chart 1: Demand vs Stock (Bar Chart)
- **Type**: Streamlit bar_chart
- **Data**: Aggregated by product
- **Series**: Units_Sold, Current_Stock
- **Layout**: 2/3 width in row 1

### Chart 2: Category Distribution (Donut Chart)
- **Type**: Matplotlib pie chart with hole
- **Data**: Category sales totals
- **Theme**: Dark (#0e1117 background)
- **Layout**: 1/3 width in row 1

### Chart 3: Sales Trend (Line Chart)
- **Type**: Streamlit line_chart
- **Data**: Daily aggregated sales
- **X-axis**: Date
- **Y-axis**: Units_Sold

### Chart 4: Price Comparison (Line Chart)
- **Type**: Streamlit line_chart
- **Data**: Product-level averages
- **Series**: Price, Competitor_Price
- **Sort**: By Price ascending

## Styling and Theming

### Color Palette
```python
CRITICAL_RED = "#ff4b4b"
HIGH_ORANGE = "#ff914d"
LOW_GREEN = "#00cc66"
DARK_BG = "#111827"
BORDER_GRAY = "#1f2937"
```

### Custom CSS (Import Box)
```css
background: #111827;
padding: 18px;
border-radius: 12px;
border: 1px solid #1f2937;
```

### Pandas Styling
```python
def highlight_risk(val):
    if val == "CRITICAL":
        return "background-color: #ff4b4b; color:white;"
    elif val == "HIGH":
        return "background-color: #ff914d; color:white;"
    elif val == "LOW":
        return "background-color: #00cc66; color:white;"
```

## API Integration Design

### Groq API Configuration
```python
Model: llama-3.1-8b-instant
Temperature: 0.5 (insights), 0.7 (copilot)
Max Tokens: 400 (insights), 500 (copilot)
```

### API Key Management
- Stored in .env file
- Loaded via environment variables
- Separate keys for different modules

### Error Handling
- API key validation
- Network timeout handling
- Rate limit management
- Graceful fallback responses

## Performance Optimization

### Data Processing
- Groupby operations for product-level aggregation
- Transform for efficient rolling calculations
- Limit context to 100 rows for AI queries
- Cap scores at 100 to prevent overflow

### UI Optimization
- Toggleable data table to reduce initial load
- Lazy loading for visualizations
- Session state for data persistence
- Efficient filtering with pandas

## Security Design

### API Key Management
- Store in .env file (not committed)
- Use environment variables
- Separate keys per service
- Rotate keys periodically

### Data Privacy
- Session-based storage only
- No disk persistence
- No third-party data sharing
- Clear session on browser close

## Deployment Considerations

### Environment Setup
```bash
pip install -r requirements.txt
streamlit run app.py
```

### Configuration Files
- .env: API keys
- requirements.txt: Dependencies
- No additional config needed

### Dependencies
```
pandas
numpy
scikit-learn
matplotlib
streamlit
groq
altair (optional)
```

## Future Design Enhancements

### UI/UX Improvements
- Dark/light theme toggle
- Customizable dashboard layouts
- Drag-and-drop file upload
- Real-time data refresh
- Export to PDF/Excel
- Mobile-responsive design

### Advanced Features
- Database integration (PostgreSQL)
- User authentication
- Multi-tenant support
- Advanced filtering and drill-downs
- Predictive analytics (LSTM, Prophet)
- Email/SMS alerts
- API endpoints for integration

### Performance
- Caching layer (Redis)
- Async processing
- Batch operations
- Incremental data loading
- Query optimization

## Testing Strategy

### Unit Tests
- Decision logic validation
- Data transformation accuracy
- API integration mocking

### Integration Tests
- End-to-end workflow
- File upload and processing
- Tab navigation
- Search functionality

### UI Tests
- Layout responsiveness
- Color scheme consistency
- Interactive element behavior
- Error message display
