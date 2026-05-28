# MOSIS Data Reconciliation Application

A modern, fast, completely client-side Single Page Application (SPA) designed to help school district administrators reconcile their end-of-year June Student Course Completion files with their beginning-of-year October Course and Student Assignment files. 

Built using **Vue.js 3**, **Vite**, and **Vuetify**, this application processes CSV files entirely in the browser to maintain strict data privacy—no data is ever sent to a server.

## Features

- **Local Data Processing:** Upload and parse large `.csv`, `.txt`, and `.tsv` files instantly and securely in your browser using PapaParse.
- **Dynamic Discrepancy Dashboards:**
  - **Missing from Assignment:** Automatically identifies records present in the June Completion dataset that are missing matching composite keys in the October files.
  - **Missing from Completion:** Flags student course assignments that were recorded in October but never officially completed in June.
- **Interactive Data Tables:**
  - View all parsed data and discrepancy reports in sleek, modern tables.
  - **Inline Editing:** Click any cell to easily fix typos or incorrect identification numbers.
  - **Add Records:** Insert missing rows seamlessly with a click of a button.
- **Export & Save:** Securely download the corrected data directly back to a CSV file for your records.

## Tech Stack
- [Vue.js 3](https://vuejs.org/)
- [Vite](https://vitejs.dev/)
- [Vuetify 3](https://vuetifyjs.com/)
- [PapaParse](https://www.papaparse.com/)

## Getting Started

### Prerequisites
- Node.js (v18 or higher recommended)
- npm

### Installation

1. **Clone the repository:**
   ```bash
   git clone <your-repository-url>
   cd "MO Course Tools"
   ```

2. **Install dependencies:**
   ```bash
   npm install
   ```

3. **Run the development server:**
   ```bash
   npm run dev
   ```
   Open `http://localhost:5173` in your browser to start reconciling your files.

4. **Build for production:**
   ```bash
   npm run build
   ```
   This will generate a `dist` directory with your compiled, production-ready static files.

## How it Works (Matching Logic)

The reconciliation engine matches records across the October and June files using specific primary keys defined by MOSIS:

- **Course Level Matching:**
  - `CurrentSchoolYear`
  - `ReportingDistrictCode`
  - `ReportingSchoolCode`
  - `EDSSN`
  - `PosCode`
  - `CTEProgType`
  - `AssignNum`

- **Student Level Matching:**
  - Uses all Course Level keys + `StateID`.

If these precise headers vary slightly in your exports, the application safely normalizes them to perform exact checks.
