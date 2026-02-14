# Requirements Document

## Project Overview
AI Retail Decision Intelligence Platform - A modern, data-driven decision support system for retail shop owners to optimize inventory management, pricing strategies, and business operations using AI-powered insights with an enhanced user interface.

## Functional Requirements

### 1. Data Management
- **FR1.1**: System shall accept CSV file uploads through a compact header import box
- **FR1.2**: System shall validate presence of required columns: Product, Category, Units_Sold, Current_Stock, Price, Competitor_Price
- **FR1.3**: System shall store uploaded data in session state for analysis
- **FR1.4**: System shall handle date parsing and time-series data
- **FR1.5**: System shall persist uploaded data across tab navigation

### 2. Analytics Dashboard
- **FR2.1**: Display KPI metrics in a 3-column layout (Total Rows, Unique Products, Categories)
- **FR2.2**: Provide toggleable raw data table view
- **FR2.3**: Visualize demand vs current stock by product (bar chart)
- **FR2.4**: Display category sales distribution as donut chart with dark theme
- **FR2.5**: Show daily sales trends over time (line chart)
- **FR2.6**: Compare product prices vs competitor prices (line chart)
- **FR2.7**: Use responsive 2-column layout for visualizations
- **FR2.8**: Include visual dividers between dashboard sections

### 3. AI Decision Intelligence
- **FR3.1**: Generate automated business decisions for each product
- **FR3.2**: Classify decisions into three categories: DISCOUNT, RESTOCK, MAINTAIN
- **FR3.3**: Provide reasoning for each decision
- **FR3.4**: Assign risk levels: HIGH, CRITICAL, LOW
- **FR3.5**: Calculate dynamic priority scores (0-100) based on gap ratio
- **FR3.6**: Display business impact assessment for each recommendation
- **FR3.7**: Present decisions in searchable, styled dataframe format
- **FR3.8**: Apply color-coded risk highlighting (red for CRITICAL, orange for HIGH, green for LOW)
- **FR3.9**: Support product search/filtering in decision table

### 4. Demand Prediction
- **FR4.1**: Calculate predicted demand using 7-day rolling average per product group
- **FR4.2**: Use predicted demand for decision-making logic
- **FR4.3**: Handle edge cases with minimum periods for rolling calculations
- **FR4.4**: Apply groupby transformation for accurate product-level predictions

### 5. AI Copilot
- **FR5.1**: Accept natural language questions from users
- **FR5.2**: Provide context-aware answers based on uploaded data
- **FR5.3**: Generate business recommendations without code
- **FR5.4**: Integrate with Groq API for LLM capabilities
- **FR5.5**: Display responses in conversational format

### 6. AI Insights Generation
- **FR6.1**: Generate overall business health score (0-100)
- **FR6.2**: Identify key business risks
- **FR6.3**: Highlight business opportunities
- **FR6.4**: Recommend actionable steps
- **FR6.5**: Analyze high stock and low stock items
- **FR6.6**: Display insights on-demand via button trigger

## Non-Functional Requirements

### 1. Performance
- **NFR1.1**: Dashboard shall load within 3 seconds for datasets up to 10,000 rows
- **NFR1.2**: AI insights generation shall complete within 5 seconds
- **NFR1.3**: Copilot responses shall be delivered within 3 seconds
- **NFR1.4**: Decision table filtering shall be instantaneous

### 2. Usability
- **NFR2.1**: Interface shall use wide layout for better data visualization
- **NFR2.2**: System shall provide clear error messages for missing columns
- **NFR2.3**: Visual indicators (colors) shall distinguish risk levels
- **NFR2.4**: Navigation shall use tabbed interface with emoji icons
- **NFR2.5**: Header shall include compact import box for easy file upload
- **NFR2.6**: Charts shall use dark theme consistent with Streamlit
- **NFR2.7**: Data table shall be toggleable to reduce visual clutter

### 3. Reliability
- **NFR3.1**: System shall handle missing data gracefully
- **NFR3.2**: System shall validate data integrity before processing
- **NFR3.3**: API failures shall be handled with appropriate error messages
- **NFR3.4**: Session state shall persist across tab switches

### 4. Security
- **NFR4.1**: API keys shall be stored in environment variables (.env file)
- **NFR4.2**: Uploaded data shall remain in user session only
- **NFR4.3**: No data shall be persisted to disk without user consent
- **NFR4.4**: API keys shall not be exposed in client-side code

### 5. Scalability
- **NFR5.1**: System shall support datasets with up to 100,000 rows
- **NFR5.2**: Aggregation logic shall be optimized for large datasets
- **NFR5.3**: Groupby operations shall be efficient for product-level analysis

## Technical Requirements

### 1. Technology Stack
- **TR1.1**: Python 3.8+ as primary programming language
- **TR1.2**: Streamlit for web interface
- **TR1.3**: Pandas for data manipulation
- **TR1.4**: Scikit-learn for machine learning capabilities
- **TR1.5**: Groq API for LLM integration
- **TR1.6**: Matplotlib for custom visualizations
- **TR1.7**: Altair for advanced charting (optional)

### 2. Data Format
- **TR2.1**: Input format: CSV files
- **TR2.2**: Required columns: Product, Category, Units_Sold, Current_Stock, Price, Competitor_Price
- **TR2.3**: Optional columns: Date (for time-series analysis)
- **TR2.4**: Date format: ISO 8601 compatible

### 3. API Integration
- **TR3.1**: Groq API model: llama-3.1-8b-instant
- **TR3.2**: Temperature: 0.5-0.7 for balanced creativity
- **TR3.3**: Max tokens: 400-500 for concise responses
- **TR3.4**: API key management via environment variables

### 4. UI Components
- **TR4.1**: Streamlit columns for responsive layouts
- **TR4.2**: Custom HTML/CSS for styled import box
- **TR4.3**: Matplotlib figures with dark theme
- **TR4.4**: Pandas styling for conditional formatting

## Business Rules

### 1. Decision Logic (Enhanced)
- **BR1.1**: RESTOCK (CRITICAL) when Current_Stock < Predicted_Demand × 0.7
- **BR1.2**: RESTOCK (HIGH) when Current_Stock < Predicted_Demand
- **BR1.3**: DISCOUNT (HIGH) when Current_Stock > Predicted_Demand × 2
- **BR1.4**: MAINTAIN (LOW) when stock is aligned with demand
- **BR1.5**: Priority scores calculated dynamically: base_score + (gap_ratio × multiplier)
- **BR1.6**: Priority scores capped at 100

### 2. Risk Assessment
- **BR2.1**: CRITICAL risk for severe understock (< 70% of demand)
- **BR2.2**: HIGH risk for moderate understock or severe overstock
- **BR2.3**: LOW risk for balanced inventory

### 3. Gap Ratio Calculation
- **BR3.1**: gap = Current_Stock - Predicted_Demand
- **BR3.2**: gap_ratio = |gap| / (Predicted_Demand + 1)
- **BR3.3**: Higher gap_ratio increases priority score

## UI/UX Requirements

### 1. Layout Structure
- **UX1.1**: Header with title, caption, and compact import box
- **UX1.2**: 3-tab navigation: Dashboard, AI Decisions, AI Copilot
- **UX1.3**: KPI metrics displayed prominently at top of dashboard
- **UX1.4**: Visual dividers between sections

### 2. Color Scheme
- **UX2.1**: CRITICAL risk: #ff4b4b (red)
- **UX2.2**: HIGH risk: #ff914d (orange)
- **UX2.3**: LOW risk: #00cc66 (green)
- **UX2.4**: Background: #111827 (dark gray)
- **UX2.5**: Border: #1f2937 (lighter gray)

### 3. Interactive Elements
- **UX3.1**: Toggle for raw data visibility
- **UX3.2**: Search box for decision filtering
- **UX3.3**: Button for AI insights generation
- **UX3.4**: Text input for AI copilot queries

## Constraints
- **C1**: Requires active internet connection for AI features
- **C2**: Requires valid Groq API key
- **C3**: Limited to CSV file format for data input
- **C4**: Demand prediction requires minimum 7 data points for accuracy
- **C5**: Context limited to 100 rows for AI copilot queries

## Future Enhancements
- Multi-file upload support
- Database integration for persistent storage
- Advanced ML models for demand forecasting (LSTM, Prophet)
- Real-time data synchronization
- Export functionality for decisions and insights (PDF, Excel)
- User authentication and multi-tenant support
- Mobile-responsive design optimization
- Integration with POS systems
- Email/SMS alerts for critical decisions
- Historical trend analysis and forecasting
- Customizable decision thresholds
- Multi-language support
