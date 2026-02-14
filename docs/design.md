# RadVisual AI: Design Document

## System Architecture

### High-Level Architecture

```
┌─────────────────┐
│  User Interface │
│   (React App)   │
└────────┬────────┘
         │
         ▼
┌─────────────────────────────────────────┐
│         API Gateway / Backend           │
│            (FastAPI/Flask)              │
└────────┬────────────────────────────────┘
         │
         ├──────────────┬──────────────┬──────────────┐
         ▼              ▼              ▼              ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│  NLP Engine  │ │   Visual     │ │  Translation │ │   Output     │
│   (spaCy/    │ │   Mapping    │ │    Engine    │ │  Generator   │
│  Transformers)│ │   Service    │ │              │ │  (PDF/Web)   │
└──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘
         │              │              │              │
         └──────────────┴──────────────┴──────────────┘
                        │
                        ▼
         ┌──────────────────────────────┐
         │   3D Asset Library (CDN)     │
         │   Medical Term Database      │
         └──────────────────────────────┘
```

### Data Flow

**User Upload → AI Parsing (NER) → Visual Mapping → PDF/Web Output**

1. **Upload Phase**: User uploads radiology report (PDF/DOCX/TXT)
2. **Extraction Phase**: OCR (if needed) + text extraction
3. **NLP Phase**: Named Entity Recognition identifies medical terms
4. **Mapping Phase**: Terms matched to 3D visual assets + simplified translations
5. **Generation Phase**: Visual report compiled with split-screen layout
6. **Delivery Phase**: Interactive web view + downloadable PDF

### Component Details

#### 1. NLP Engine
**Technology:** Python + spaCy + BioBERT/ClinicalBERT

**Responsibilities:**
- Medical Named Entity Recognition (NER)
- Anatomical structure identification
- Pathology classification
- Severity extraction

**Pipeline:**
```python
Report Text → Tokenization → NER Model → Entity Linking → Structured Output
```

**Output Format:**
```json
{
  "findings": [
    {
      "term": "osteophytic lipping",
      "category": "pathology",
      "anatomy": "vertebral body",
      "location": "L4-L5",
      "severity": "mild"
    }
  ]
}
```

#### 2. Visual Mapping Service
**Technology:** Python + Custom Mapping Engine

**Responsibilities:**
- Match medical terms to 3D asset IDs
- Determine optimal anatomical view angles
- Generate annotation coordinates
- Handle multiple findings on single anatomy

**Mapping Database Schema:**
```
medical_terms:
  - term_id
  - medical_name
  - simple_name
  - asset_id (FK to 3D models)
  - body_system
  - anatomical_region
```

#### 3. Translation Engine
**Technology:** Python + Custom Rule-Based System + GPT-4 API (fallback)

**Responsibilities:**
- Convert medical jargon to Grade 6-8 language
- Add contextual explanations
- Generate severity indicators
- Create "Learn More" content

**Translation Strategy:**
- Primary: Pre-built translation dictionary (5000+ terms)
- Secondary: Template-based generation
- Fallback: GPT-4 API with medical prompt engineering

#### 4. 3D Asset Library
**Technology:** Three.js + Blender-exported GLB/GLTF models

**Asset Organization:**
```
/assets/
  /skeletal/
    - spine_lumbar.glb
    - femur.glb
  /cardiovascular/
    - heart_normal.glb
    - heart_enlarged.glb
  /respiratory/
    - lungs_normal.glb
    - lungs_nodule.glb
  /annotations/
    - highlight_red.png
    - arrow_indicator.png
```

**Model Specifications:**
- Format: GLB (optimized GLTF)
- Polygon count: <50k per model
- Texture resolution: 2048x2048
- File size: <5MB per asset

## UI/UX Design

### Design Philosophy: Clinical Minimalist

**Principles:**
- Clean, uncluttered interface
- High contrast for readability
- Large, accessible typography
- Calming color palette (blues, whites, soft grays)
- Progressive disclosure of information

### Visual Strategy: Split-Screen Layout

```
┌─────────────────────────────────────────────────────────┐
│                    RadVisual AI                         │
│                  [Logo] [Help] [Download]               │
├──────────────────────────┬──────────────────────────────┤
│                          │                              │
│   PROFESSIONAL REPORT    │   VISUAL PATIENT STORY       │
│                          │                              │
│  [Original Report Text]  │   [3D Anatomical Model]      │
│                          │                              │
│  "Findings:              │   ┌──────────────────────┐   │
│   1. Mild degenerative   │   │   [Interactive 3D]   │   │
│      disc disease at     │   │   Spine Model with   │   │
│      L4-L5 with          │   │   Highlighted L4-L5  │   │
│      osteophytic         │   └──────────────────────┘   │
│      lipping..."         │                              │
│                          │   What This Means:           │
│  [Scroll for more]       │   • Wear and tear of the     │
│                          │     cushion between your     │
│                          │     lower back bones         │
│                          │   • Severity: MILD           │
│                          │   • Small bone spurs         │
│                          │                              │
│                          │   [Learn More ▼]             │
│                          │                              │
└──────────────────────────┴──────────────────────────────┘
```

### Key Screens

#### 1. Upload Screen
- Large drag-and-drop zone
- Supported format icons (PDF, DOCX, TXT)
- Privacy assurance message
- Sample report link for demo

#### 2. Processing Screen
- Animated progress indicator
- Step-by-step status:
  - ✓ Reading report
  - ✓ Identifying medical terms
  - ✓ Creating visuals
  - ✓ Generating your report

#### 3. Visual Report Screen (Main Interface)
**Left Panel: Professional Report**
- Original report text
- Highlighted medical terms (hover for quick definition)
- Scrollable with sticky header
- Print-friendly view toggle

**Right Panel: Visual Patient Story**
- 3D anatomical model (interactive)
  - Rotate: Click and drag
  - Zoom: Scroll wheel
  - Reset view: Double-click
- Simplified findings cards
- Color-coded severity indicators:
  - 🟢 Mild (Green)
  - 🟡 Moderate (Yellow)
  - 🔴 Severe (Red)
- Expandable "Learn More" sections
- Related anatomy information

#### 4. Export Options
- Download PDF button
- Share link (24-hour expiration)
- Print-optimized view
- Email to doctor option

### Typography

**Font Family:** Inter (primary), System UI (fallback)

**Scale:**
- H1 (Page Title): 32px, Bold
- H2 (Section Headers): 24px, Semibold
- H3 (Finding Titles): 20px, Medium
- Body (Simplified Text): 18px, Regular
- Small (Metadata): 14px, Regular

**Accessibility:**
- Minimum contrast ratio: 4.5:1
- Scalable up to 200% without layout breaking
- Large text mode: +4px to all sizes

### Color Palette

**Primary Colors:**
- Clinical Blue: #2563EB (buttons, links)
- Trust Navy: #1E3A8A (headers)
- Clean White: #FFFFFF (backgrounds)
- Soft Gray: #F3F4F6 (secondary backgrounds)

**Semantic Colors:**
- Success Green: #10B981 (mild findings)
- Warning Yellow: #F59E0B (moderate findings)
- Alert Red: #EF4444 (severe findings)
- Info Blue: #3B82F6 (informational elements)

**Text Colors:**
- Primary Text: #111827
- Secondary Text: #6B7280
- Disabled Text: #9CA3AF

### Interactive Elements

**3D Model Controls:**
- Orbit: Left-click drag
- Pan: Right-click drag
- Zoom: Scroll wheel
- Reset: "Reset View" button
- Preset views: Front, Side, Top buttons

**Annotations:**
- Pulsing highlight on affected areas
- Clickable hotspots for detailed info
- Tooltip on hover with term name

### Responsive Design

**Desktop (1920x1080):** Full split-screen layout
**Tablet (768x1024):** Stacked layout with tabs
**Mobile (375x667):** Single column, swipeable cards

## Technical Stack

### Frontend
**Framework:** React 18+ with TypeScript

**Key Libraries:**
- Three.js (3D rendering)
- React Three Fiber (React + Three.js integration)
- Tailwind CSS (styling)
- Framer Motion (animations)
- React PDF (PDF generation)
- Zustand (state management)

**Structure:**
```
/src
  /components
    - UploadZone.tsx
    - SplitView.tsx
    - ThreeDViewer.tsx
    - FindingCard.tsx
  /services
    - api.ts
    - pdfGenerator.ts
  /hooks
    - useReportProcessing.ts
  /types
    - report.types.ts
```

### Backend
**Framework:** FastAPI (Python 3.10+)

**Key Libraries:**
- spaCy + scispaCy (NLP)
- transformers (BioBERT)
- PyPDF2 (PDF parsing)
- pytesseract (OCR)
- Pillow (image processing)
- pydantic (data validation)

**Structure:**
```
/backend
  /api
    - routes.py
  /services
    - nlp_service.py
    - mapping_service.py
    - translation_service.py
  /models
    - report_model.py
  /utils
    - ocr.py
    - validators.py
```

### 3D Assets Pipeline
**Creation:** Blender 3.6+

**Workflow:**
1. Model anatomy in Blender (or source from open libraries)
2. UV unwrap and texture
3. Optimize geometry (<50k polygons)
4. Export as GLB with Draco compression
5. Host on CDN (Cloudflare/AWS CloudFront)

**Asset Management:**
- Version control for 3D files (Git LFS)
- Automated optimization pipeline
- CDN caching strategy

### Infrastructure
**Hosting:** 
- Frontend: Vercel / Netlify
- Backend: AWS Lambda + API Gateway (serverless)
- 3D Assets: AWS S3 + CloudFront CDN
- Database: PostgreSQL (AWS RDS) for term mappings

**CI/CD:**
- GitHub Actions for automated testing
- Automated deployment on merge to main
- Staging environment for testing

## Data Models

### Report Entity
```typescript
interface Report {
  id: string;
  uploadedAt: Date;
  originalText: string;
  modality: 'CT' | 'MRI' | 'X-Ray' | 'Ultrasound';
  findings: Finding[];
  processedAt: Date;
  expiresAt: Date;
}
```

### Finding Entity
```typescript
interface Finding {
  id: string;
  medicalTerm: string;
  simplifiedText: string;
  anatomy: string;
  location: string;
  severity: 'mild' | 'moderate' | 'severe';
  assetId: string;
  annotationCoords: {x: number, y: number, z: number};
  learnMoreContent: string;
}
```

### Visual Asset Entity
```typescript
interface VisualAsset {
  id: string;
  name: string;
  bodySystem: string;
  anatomicalRegion: string;
  fileUrl: string;
  thumbnailUrl: string;
  polygonCount: number;
  fileSize: number;
}
```

## Security & Privacy

### Data Handling
- No user accounts or authentication
- Reports processed in-memory when possible
- Temporary storage encrypted (AES-256)
- Automatic deletion after 24 hours
- No analytics or tracking of medical data

### Compliance
- HIPAA-inspired practices (not HIPAA-certified for competition)
- Data processing agreement templates
- Privacy policy clearly displayed
- Disclaimer: "Educational tool, not for diagnosis"

## Performance Optimization

### Frontend
- Code splitting by route
- Lazy loading of 3D models
- Image optimization (WebP format)
- Service worker for offline capability
- Memoization of expensive computations

### Backend
- Caching of common term translations
- Batch processing for multiple findings
- Async processing with job queue
- Database query optimization
- CDN for static assets

### 3D Rendering
- Level of Detail (LOD) for models
- Frustum culling
- Texture compression
- Progressive loading

## Testing Strategy

### Unit Tests
- NLP extraction accuracy
- Translation quality
- Visual mapping correctness

### Integration Tests
- End-to-end report processing
- API endpoint validation
- 3D asset loading

### User Testing
- Comprehension testing with patients
- Accessibility testing with screen readers
- Usability testing with elderly users

## Deployment Plan

### Phase 1: MVP (Competition Demo)
- Core NLP engine with 1000 terms
- 50 essential 3D models
- Basic split-screen interface
- PDF export

### Phase 2: Beta Release
- Expanded term library (5000+)
- 200+ 3D models
- Mobile responsive design
- Audio narration

### Phase 3: Production
- Multi-language support
- Advanced 3D interactions
- EHR integration APIs
- Analytics dashboard for providers

## Success Metrics

### User Experience
- Time to comprehension: <5 minutes
- User satisfaction: 4.5/5 stars
- Accessibility score: WCAG AA compliance

### Technical Performance
- Processing time: <15 seconds
- 3D load time: <2 seconds
- Uptime: 99.5%

### Medical Accuracy
- Term extraction: 95%+ accuracy
- Anatomical correctness: 99%+
- Translation validation by medical professionals

## Future Enhancements

- AR visualization via mobile camera
- Voice-guided report walkthrough
- Integration with patient portals
- Multi-language support (Spanish, Mandarin)
- Comparison view for follow-up scans
- Printable patient education materials
