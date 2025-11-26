# Competitor-Dashboard
Python code for websrcaping of news articles pertaining to key competitors, market and industry
import feedparser
import pandas as pd
from bs4 import BeautifulSoup
from datetime import datetime
from urllib.parse import quote

OUTPUT_EXCEL = "news_output.xlsx"
last_days = 7

# -------------------------------
# 1️⃣ SBU → Keyword Master Mapping
# -------------------------------
SBU_KEYWORD_MAP = {
    "India T&D": [
          # Transmission & Distribution Fundamentals
    "transmission", "distribution", "T&D", "power grid", "electricity grid",
    "substation", "switchyard", "switching station", "pooling station",
    
    # Transmission System Types
    "ISTS", "Inter-State Transmission System", "Intra-STS", 
    "Intra-State Transmission System", "inter-regional transmission",
    "intra-state transmission", "inter-state transmission",
    
    # Equipment - Circuit Breakers & Switchgear
    "circuit breaker", "GIS", "Gas Insulated Switchgear", 
    "AIS", "Air Insulated Switchgear", "MTS", "Mobile Transformer Substation",
    "hybrid switchgear", "disconnect switch", "disconnector",
    "isolator", "bus isolator", "line isolator", "earth switch",
    
    # Transformers & Capacity
    "transformer", "power transformer", "auto-transformer",
    "MVA", "MW capacity", "GVA", "transformation capacity",
    "step-up transformer", "step-down transformer",
    
    # Transmission Lines & Conductor
    "transmission line", "distribution line", "overhead line", "OHL",
    "underground cable", "circuit km", "ckm", "circuit kilometer",
    "tower", "pylon", "lattice tower", "monopole", "pole",
    "conductor", "ACSR", "AAAC", "ACCC", "bundle conductor",
    "quad conductor", "twin conductor", "single conductor",
    
    # Insulation & Protection
    "insulator", "disc insulator", "polymer insulator", "composite insulator",
    "lightning arrester", "surge arrester", "LA",
    "earthing", "grounding", "earth mat", "ground wire",
    
    # OPGW & Communication
    "OPGW", "Optical Ground Wire", "optical fiber", "fibre optic",
    "communication system", "SCADA", "telecom infrastructure",
    
    # Voltage Levels & Technology
    "HVDC", "High Voltage DC", "AC transmission", "HVAC", "high voltage AC",
    "765 kV", "400 kV", "220 kV", "132 kV", "110 kV", "66 kV",
    "EHV", "extra high voltage", "UHV", "ultra high voltage",
    "1200 kV", "±800 kV HVDC", "bipole", "monopole HVDC",
    "medium voltage", "low voltage", "11 kV", "33 kV",
    
    # Bay & Busbar Configuration
    "bay", "feeder bay", "line bay", "transformer bay", "bus coupler bay",
    "busbar", "bus bar", "main bus", "transfer bus", "auxiliary bus",
    "double busbar", "single busbar", "ring bus", "mesh bus",
    "bus sectionalizer", "bus coupler", "TBC", "bypass isolator",
    
    # Grid Operations & Quality
    "power evacuation", "evacuation scheme", "grid connectivity",
    "grid stability", "grid balancing", "grid modernization", "grid digitalization",
    "grid automation", "grid access", "connectivity",
    "power quality", "reactive power", "VAR compensation",
    "harmonic distortion", "frequency regulation", "voltage regulation",
    "transmission losses", "technical losses", "AT&C losses",
    
    # Smart Grid & Metering
    "Advanced Metering Infrastructure", "AMI", "smart meter", "smart grid",
    "energy meter", "ABT meter", "availability based tariff",
    "metering", "protection relay", "numerical relay",
    
    # Integration & Storage
    "renewable energy integration", "RE integration", "solar integration", 
    "wind integration", "hybrid transmission",
    "battery storage", "BESS", "Battery Energy Storage System",
    "energy storage system", "pumped storage",
    "ancillary services", "spinning reserve",
    
    # Project Specifications
    "transmission capacity", "substation capacity", "capacity addition",
    "transmission expansion", "network expansion",
    "GW", "gigawatt", "megawatt", "MW",
    "last-mile connectivity", "distribution infrastructure",
    "transmission infrastructure",
    
    # Civil & MEP Works
    "civil works", "foundation", "civil construction",
    "MEP", "Mechanical Electrical Plumbing",
    "substation building", "control room", "relay room",
    
    # Project Documentation
    "DPR", "Detailed Project Report", "feasibility study",
    "technical specification", "performance standards",
    "SLD", "Single Line Diagram", "GA", "General Arrangement",
    "GTP", "Guaranteed Technical Particulars",
    "switching scheme", "protection scheme",
    
    # Regional Coverage
    "Northern Region", "Southern Region", "Western Region", 
    "Eastern Region", "North Eastern Region", "NER",
    "Delhi", "Mumbai", "Chennai", "Kolkata", "Bangalore", 
    "Hyderabad", "Pune", "Ahmedabad", "Jaipur", "Lucknow",
    
    # Regulatory & Planning Bodies
    "PowerGrid", "PGCIL", "Power Grid Corporation of India",
    "CEA", "Central Electricity Authority",
    "CERC", "Central Electricity Regulatory Commission",
    "NTPC", "National Thermal Power Corporation",
    "CTU", "Central Transmission Utility",
    "STU", "State Transmission Utility", "STTU",
    "distribution utility", "DISCOM",
    "RPC", "Regional Power Committee",
    "NLDC", "National Load Dispatch Centre",
    "RLDC", "Regional Load Dispatch Centre",
    
    # Planning & Policies
    "National Electricity Plan", "NEP", "transmission plan",
    "Green Energy Corridor", "GEC", "renewable energy transmission",
    "REZ", "Renewable Energy Zone",
    "National Grid", "integrated grid", "synchronous grid",
    
    # Bidding & Contracting Terms
    "EPC contract", "EPC", "Engineering Procurement Construction",
    "TBCB", "Tariff Based Competitive Bidding",
    "tender", "RFQ", "Request for Quotation",
    "RFP", "Request for Proposal", "RfP",
    "bid", "bidding process", "competitive bidding",
    "award", "LOA", "Letter of Award",
    "LOI", "Letter of Intent", "execution",
    "commissioning", "COD", "Commercial Operation Date",
    "SCOD", "Scheduled COD",
    "BPC", "Bid Process Coordinator",
    "TSP", "Transmission Service Provider",
    "SPV", "Special Purpose Vehicle",
    "TSA", "Transmission Service Agreement",
    "BEC", "Bid Evaluation Committee",
    "e-RA", "e-reverse auction", "e-auction",
    
    # Financial & Investment
    "capex", "capital expenditure", "investment",
    "transmission charges", "transmission tariff",
    "wheeling charges", "open access charges",
    "transmission license", "ISTS charges",
    
    # Testing & Quality
    "type test", "routine test", "acceptance test",
    "pre-commissioning", "testing and commissioning",
    "performance test", "reliability test",
    "FAT", "Factory Acceptance Test",
    "SAT", "Site Acceptance Test",
    
    # Access & Trading
    "open access", "MTOA", "Medium Term Open Access",
    "STOA", "Short Term Open Access", "LTOA", "Long Term Open Access",
    "LTA", "Long Term Access", "power exchange",
    "power trading", "electricity trading",
    
    # State Utilities (Major ones)
    "PGCIL", "POWERGRID",
    "NTPC", "NHPC", "THDC",
    "PSTCL", "Punjab State Transmission",
    "UPPTCL", "Uttar Pradesh Transmission",
    "MSETCL", "Maharashtra Transmission",
    "TSTRANSCO", "Telangana Transmission",
    "APTRANSCO", "Andhra Pradesh Transmission",
    "KPTCL", "Karnataka Transmission",
    "TNEB", "Tamil Nadu Electricity Board",
    "GETCO", "Gujarat Transmission",
    "RRVPNL", "Rajasthan Transmission",
    "MPPTCL", "Madhya Pradesh Transmission",
    "WBSETCL", "West Bengal Transmission",
    "OPTCL", "Odisha Transmission",
    "BSPTCL", "Bihar Transmission",
    "JKTPTCL", "Jharkhand Transmission",
    "PDD", "Power Development Department",
    
    # Financing & Development Bodies
    "REC", "Rural Electrification Corporation",
    "RECPDCL", "REC Power Development Consultancy",
    "RECTPCL", "REC Transmission Projects",
    "PFC", "Power Finance Corporation",
    "PFCCL", "PFC Consulting Limited",
    "IIFCL", "India Infrastructure Finance Company",
    
    # Make in India & Local Content
    "Make in India", "local content", "Class-I local supplier",
    "indigenous", "domestic manufacturing",
    
    # VRE & Renewables
    "VRE", "Variable Renewable Energy",
    "solar power evacuation", "wind power evacuation",
    "renewable energy evacuation", "green energy",
    "solar park", "wind farm", "renewable park",
    
    # Project Execution
    "project execution", "milestone", "progress report",
    "project monitoring", "Independent Engineer", "IE",
    "project delay", "time extension", "project schedule",
    "GANTT chart", "critical path",
    
    # Technology & Innovation
    "digital substation", "IEC 61850", "process bus",
    "online monitoring", "condition monitoring",
    "asset management", "life cycle management",
    "reactive compensation", "SVC", "STATCOM",
    "FACTS", "Flexible AC Transmission System"
    ],

    "International T&D": [
            # Core International T&D Terms
    "transmission", "international transmission", "cross-border transmission",
    "transmission infrastructure", "transmission line", "overhead transmission",
    "underground transmission", "transmission corridor", "transmission project",
    "power transmission", "international order", "overseas order", "export order",
    "overseas project", "international project", "global project",
    
    # International EPC & Contracting
    "EPC contract", "international EPC", "overseas EPC",
    "EPCM", "Engineering Procurement Construction Management",
    "turnkey project", "complete package", "turnkey contract",
    "engineering procurement construction",
    "global EPC contractor", "international contractor",
    "joint venture", "consortium", "partnership", "subcontract",
    "competitive bidding", "international tender", "international bid",
    "contract negotiation", "performance guarantee",
    "liquidated damages", "LD", "completion deadline",
    "contract value", "order value",
    
    # Substation & Equipment
    "substation", "transmission substation", "converter station",
    "transformer station", "dispatch center", "control center",
    "GIS", "Gas Insulated Switchgear", "AIS", "Air Insulated Switchgear",
    "circuit breaker", "disconnect switch", "disconnector",
    "isolator", "earthing switch", "earth switch",
    "current transformer", "CT", "potential transformer", "PT",
    "protection equipment", "protection relay", "numerical relay",
    "surge arrester", "lightning arrester", "LA",
    "busbar", "bus bar", "main bus", "transfer bus",
    "bay", "feeder bay", "line bay", "transformer bay",
    "breaker and a half", "double busbar", "single busbar", "ring bus",
    
    # Power Lines & Cables
    "OHL", "overhead line", "overhead transmission line",
    "transmission line", "power line", "distribution line",
    "tower", "pylon", "lattice tower", "monopole", "pole",
    "conductor", "ACSR", "AAAC", "ACCC",
    "cable", "underground cable", "power cable",
    "subsea cable", "submarine cable", "submarine transmission",
    "HVDC cable", "HVAC cable", "AC cable", "DC cable",
    "mass-impregnated cable", "XLPE cable", "paper insulated cable",
    "optic fiber", "OPGW", "Optical Ground Wire",
    "insulator", "disc insulator", "composite insulator", "polymer insulator",
    "earthing", "grounding", "earth mat", "ground wire",
    
    # Voltage Levels & Systems
    "HVDC", "High Voltage DC", "Direct Current transmission",
    "HVAC", "high voltage AC", "AC transmission",
    "EHV", "Extra High Voltage",
    "UHV", "Ultra High Voltage", "UHVAC", "UHVDC",
    "765 kV", "500 kV", "400 kV", "380 kV", "330 kV",
    "275 kV", "220 kV", "132 kV",
    "±800 kV", "±600 kV", "±500 kV", "±400 kV HVDC",
    "megavolt transmission", "high voltage transmission",
    "voltage level", "voltage rating", "transmission voltage",
    
    # HVDC & Advanced Technology
    "HVDC link", "HVDC interconnector", "HVDC connection",
    "point-to-point HVDC", "multiterminal HVDC",
    "HVDC Light", "VSC", "Voltage Sourced Converter",
    "line commutated converter", "thyristor", "IGBT",
    "FACTS", "Flexible AC Transmission System",
    "SVC", "Static VAR Compensator", "VAR compensation",
    "STATCOM", "STATic synchronous COMpensator",
    "hybrid HVDC breaker", "HVDC breaker",
    "converter technology", "converter station",
    "AC-DC conversion", "DC grid", "multi-terminal DC",
    
    # Interconnection & Integration
    "interconnection", "interconnector", "regional interconnection",
    "bilateral interconnection", "multilateral interconnection",
    "cross-border interconnection", "cross-border power trade",
    "power trading", "power exchange", "electricity trading",
    "power import", "power export", "power sharing",
    "grid connectivity", "grid integration", "renewable integration",
    "grid access", "evacuation scheme", "connectivity scheme",
    
    # Regional Markets - Middle East & GCC
    "Middle East", "GCC", "Gulf Cooperation Council", "Gulf region",
    "Saudi Arabia", "UAE", "United Arab Emirates", "Qatar",
    "Kuwait", "Oman", "Bahrain", "Doha",
    "Riyadh", "Abu Dhabi", "Dubai", "Muscat",
    "KSA", "Kahramaa", "KAEPCO", "SAECO",
    "Saudi Electricity Company", "SEC", "Saudi Aramco",
    "Jeddah", "Mecca", "Dammam",
    
    # Regional Markets - Africa
    "Africa", "African transmission",
    # East Africa
    "East Africa", "Kenya", "Tanzania", "Uganda", "Ethiopia",
    "Kenya Electricity Transmission Company", "Ketraco",
    "Tanzania Electric Supply Company", "Tanesco",
    "Uganda Electricity Transmission Company", "UETCL",
    "East African Power Pool", "EAPP",
    "Kenya-Tanzania interconnector", "Ethiopia-Kenya interconnector",
    # West Africa
    "West Africa", "Nigeria", "Ghana", "Senegal", "Ivory Coast",
    "Transmission Company of Nigeria", "TCN", "Benin",
    "West African Power Pool", "WAPP",
    # Southern Africa
    "Southern Africa", "South Africa", "Botswana", "Namibia",
    "Mozambique", "Zambia", "Zimbabwe", "Malawi",
    "Southern African Power Pool", "SAPP", "Eskom",
    # North Africa
    "North Africa", "Egypt", "Morocco", "Algeria", "Tunisia",
    "Egyptian Electricity Transmission Company", "EETC",
    "Office National de l'Électricité", "ONE",
    
    # Regional Markets - Southeast Asia
    "Southeast Asia", "ASEAN", "ASEAN Power Grid", "APG",
    "Vietnam", "Indonesia", "Thailand", "Philippines",
    "Malaysia", "Singapore", "Myanmar",
    "Lao PDR", "Laos", "Cambodia",
    "Greater Mekong Subregion", "GMS",
    "EVN", "PT PLN", "Metropolitan Electricity Authority", "MEA",
    "National Power Company", "NPC", "Tenaga Nasional", "TNB",
    # Cross-border ASEAN Projects
    "Lao-Thailand-Malaysia-Singapore Power Integration", "LTMS-PIP",
    "ASEAN Power Grid Financing Facility", "APGF",
    
    # Regional Markets - South Asia
    "South Asia", "Sri Lanka", "Bangladesh", "Pakistan",
    "Ceylon Electricity Board", "CEB",
    "Pakistan Transmission Company", "NTDC",
    
    # Regional Markets - Central Asia
    "Central Asia", "Uzbekistan", "Kazakhstan", "Turkmenistan",
    "Tajikistan", "Kyrgyzstan",
    "Central Asian Power System", "CAPS",
    
    # Regional Markets - Europe & Eastern Europe
    "Europe", "Eastern Europe", "Central Europe",
    "Poland", "Bulgaria", "Romania", "Czech Republic",
    "Hungary", "Slovakia", "Serbia", "Croatia",
    "Baltic states", "Lithuania", "Latvia", "Estonia",
    "Greece", "Cyprus", "Italy", "Spain",
    "Black Sea interconnector", "Greece-Cyprus-Israel submarine link",
    "Great Sea Interconnector", "GECI", "submarine link",
    # European Transmission Bodies
    "ENTSO-E", "European Network of Transmission System Operators",
    "CEIC", "Central European Infrastructure Commission",
    "TSO", "Transmission System Operator",
    
    # Turkey & Transcontinental Projects
    "Turkey", "Türkiye", "TEİAŞ", "Turkish Electricity Transmission",
    "Turkey Renewable Energy Scale-Up", "ECARES",
    "Turkey HVDC corridor", "Turkey-EU interconnector",
    "Dardanelles transmission", "Asia-Europe transmission link",
    "Black Sea transmission", "Bosphorus interconnector",
    
    # Regional Markets - Latin America
    "Latin America", "South America", "Central America",
    # Brazil
    "Brazil", "Brazilian transmission", "Eletrobras",
    "Rio Madeira HVDC", "Xiangjiaba-Shanghai UHVDC",
    "Brazil transmission auction", "Brazilian grid modernization",
    # Other Latin American Countries
    "Mexico", "Argentina", "Chile", "Colombia", "Peru",
    "Peru-Chile interconnector", "InterAndes HVDC",
    "Brazil-Argentina HVDC", "Garabi HVDC",
    "Central America", "Honduras", "Nicaragua",
    
    # Regional Markets - Asia-Pacific
    "Asia-Pacific", "Asia", "Pacific Islands",
    "Australia", "New Zealand", "PNG", "Papua New Guinea",
    "Pacific Islands", "Fiji", "Solomon Islands", "Vanuatu",
    # PNG & Pacific Projects
    "PNG Power Limited", "PNG transmission", "Kimbe-Biala transmission",
    "Pacific electrification", "PNG electrification program",
    
    # Major Global Transmission Projects
    "Sodo-Moyale-Suswa transmission", "Ethiopia-Kenya interconnector",
    "NorNed", "Norway-Netherlands interconnector",
    "NEMO Link", "UK-Belgium interconnector",
    "Celtic Interconnector", "Ireland-France HVDC",
    "Seikan Tunnel", "Japan-Hokkaido interconnection",
    
    # Project Documentation & Studies
    "DPR", "Detailed Project Report",
    "feasibility study", "techno-economic feasibility",
    "environmental assessment", "environmental impact assessment", "EIA",
    "social impact assessment", "SIA",
    "project finance", "project financing",
    
    # Execution & Commissioning
    "execution", "project execution", "implementation",
    "testing", "commissioning", "operational handover",
    "FAT", "Factory Acceptance Test",
    "SAT", "Site Acceptance Test",
    "pre-commissioning", "testing and commissioning",
    "operational phase", "commercial operation",
    "COD", "Completion Date", "SCOD", "Scheduled COD",
    
    # Global EPC & Equipment Manufacturers
    # Major Global EPC Contractors
    "Hyundai E&C", "Hyundai Engineering", "Samsung Engineering",
    "Siemens", "Siemens Energy", "ABB", "Schneider Electric",
    "Technip Energies", "Worley", "Petrofac",
    "Bouygues Energies & Services", "Larsen & Toubro", "L&T",
    "NPCC", "National Petroleum Construction Company",
    "Dodsal", "Galfar Engineering",
    "National Contracting Company", "NCC",
    # Transmission Equipment Manufacturers
    "ABB transmission equipment", "Siemens transmission", "Schneider transmission",
    "Hitachi Energy", "Hitachi transmission",
    "Prysmian", "transmission cables", "submarine cables",
    "Sumitomo Electric", "cable manufacturer",
    "GE Grid Solutions", "Arteche",
    "JMC Projects India", "transmission towers",
    "Kalpataru", "transmission towers",
    
    # Standards & Compliance
    "international standards", "IEEE", "IEC",
    "CIGRE", "International Council on Large Electric Systems",
    "Indian standards", "IS code", "grid codes",
    "IEEE 1547", "IEEE 2800", "interconnection standards",
    "local regulations", "grid regulations",
    "safety standards", "international norms",
    
    # Contract Types & Delivery Models
    "design-build", "design-bid-build", "EPCM",
    "supply and installation", "design-supply-install",
    "BOT", "Build Operate Transfer",
    "availability-based tariff", "ABT",
    "PPP", "public-private partnership",
    
    # Project Types & Technology Focus
    "renewable energy transmission", "renewable integration",
    "wind power evacuation", "solar power evacuation",
    "hydro power transmission", "hydropower interconnection",
    "cross-border power trading", "bilateral power trade",
    "regional integration", "grid stabilization",
    "frequency regulation", "voltage regulation",
    "reactive power support", "black start capability",
    
    # Bidding & Order-Related Terms
    "order intake", "order book", "order value",
    "YTD", "year-to-date", "order announcement",
    "repeat order", "extension order",
    "BAG", "Bahrain, Abu Dhabi, Qatar",
    "GCC order", "Middle East order", "Africa order",
    "Americas order", "European order",
    "international revenue", "overseas revenue",
    
    # Specific Market Dynamics
    "market presence", "footprint", "country presence",
    "market entry", "new market", "established market",
    "client acquisition", "marquee client", "prestigious client",
    "first order in market", "maiden order",
    "repeat client", "long-standing client",
    "customer diversification", "geographic diversification",
    
    # Financial & Project Metrics
    "capex", "capital expenditure", "investment",
    "project cost", "contract value",
    "transmission charges", "transmission tariff",
    "wheeling charges", "open access charges",
    "financing", "project financing", "infrastructure financing",
    
    # Challenges & Regulatory
    "regulatory approval", "environmental clearance",
    "government approval", "ministerial approval",
    "land acquisition", "right of way", "RoW",
    "political risk", "geopolitical risk",
    "technical standards", "compliance requirements",
    "local content", "local partner", "local subcontractor",
    
    # Regional Trade & Power Exchange
    "electricity trading platform", "power exchange",
    "bilateral trading", "multilateral trading",
    "regional cooperation", "power pool",
    "frequency matching", "50 Hz", "60 Hz",
    "load flow analysis", "power dispatch",
    "system integration", "grid synchronization",
    
    # Submarine & Offshore Transmission
    "submarine transmission", "subsea transmission",
    "offshore cable", "submarine cable system",
    "underwater transmission", "sub-sea HVDC",
    "cable ship", "installation vessel",
    "landfall", "cable route", "cable protection",
    "marine environment", "offshore engineering",
    
    # Advanced Grid Technology
    "digital substation", "smart substation",
    "IEC 61850", "process bus", "Ethernet",
    "SCADA", "Supervisory Control and Data Acquisition",
    "EMS", "Energy Management System",
    "grid management", "network management",
    "online monitoring", "condition monitoring",
    "real-time operation", "remote operation",
    "automation", "intelligent control",
      # Cross-Border Coordination
    "bilateral agreement", "power purchase agreement", "PPA",
    "transmission service agreement", "TSA",
    "interconnection agreement",
    "regional protocol", "operating protocol",
    "grid code compliance", "technical requirements",
    "mutual cooperation", "regulatory framework",
    
    # Market Segments
    "transmission-only project", "generation-transmission",
    "greenfield project", "brownfield project",
    "retrofit", "modernization", "upgradation",
    "capacity expansion", "system strengthening",
    "congestion relief", "loss reduction"],

    "Civil": [    # Core Civil Engineering Terms
    "civil engineering", "civil infrastructure", "civil construction",
    "civil works", "civil project", "infrastructure development",
    "infrastructure project", "infrastructure EPC","engineering", "infrastructure", "construction", "project", "infrastructure development",
    
    # Roads & Highways - Core Terms
    "highway", "expressway", "motorway", "freeway", "road project",
    "national highway", "NH", "state highway", "SH",
    "district road", "village road", "rural road",
    "expressway project", "access-controlled highway",
    
    # Roads - Technical Specifications
    "lane", "carriageway", "four-lane", "six-lane", "eight-lane",
    "two-lane", "single lane", "dual carriageway",
    "paved shoulder", "unpaved shoulder", "median",
    "service road", "frontage road", "slip road",
    "bituminous road", "concrete road", "pavement",
    "flexible pavement", "rigid pavement", "WBM", "water bound macadam",
    "DBM", "dense bituminous macadam", "BM", "bituminous macadam",
    "BC", "bituminous concrete", "SDBC", "semi-dense bituminous concrete",
    "GSB", "granular sub-base", "CTB", "cement treated base",
    "PQC", "pavement quality concrete", "CC pavement",
    "white topping", "asphalt", "tar", "bitumen",
    
    # Roads - Major Programs & Projects
    "Bharatmala", "Bharatmala Pariyojana", "Golden Quadrilateral", "GQ",
    "North-South Corridor", "East-West Corridor", "NSEW",
    "NHDP", "National Highways Development Project",
    "Gati Shakti", "PM Gati Shakti", "National Master Plan",
    "SARDP-NE", "Special Accelerated Road Development Programme",
    "Setu Bharatam", "Setu Bharatam Programme",
    "Delhi-Mumbai Expressway", "Mumbai-Nagpur Expressway",
    "Purvanchal Expressway", "Bundelkhand Expressway",
    "coastal road", "coastal highway", "ring road",
    "peripheral expressway", "orbital highway",
    
    # Bridges - Types & Components
    "bridge", "viaduct", "overpass", "underpass", "flyover",
    "grade separator", "ROB", "Road Over Bridge",
    "RUB", "Road Under Bridge", "RoB", "RuB",
    "level crossing", "LC", "railway crossing",
    "rail flyover", "rail over rail bridge",
    "suspension bridge", "cable-stayed bridge", "cantilever bridge",
    "arch bridge", "truss bridge", "beam bridge", "girder bridge",
    "box girder", "prestressed girder", "precast girder",
    "pier", "abutment", "deck", "superstructure", "substructure",
    "bearing", "expansion joint", "parapet", "crash barrier",
    "approach road", "approach slab",
    
    # Tunnels - Types & Methods
    "tunnel", "underground tunnel", "road tunnel", "rail tunnel",
    "excavation", "boring", "tunneling", "tunnelling",
    "TBM", "Tunnel Boring Machine", "EPB", "Earth Pressure Balance",
    "NATM", "New Austrian Tunneling Method",
    "cut-and-cover", "cut and cover", "cover-and-cut",
    "drilling and blasting", "D&B method",
    "shield tunneling", "fore poling", "pipe roofing",
    "shotcrete", "rock bolting", "lattice girder",
    "primary lining", "secondary lining", "tunnel lining",
    "ventilation shaft", "escape shaft", "cross passage",
    "portal", "tunnel portal", "approach tunnel",
    
    # Water Infrastructure
    "dam", "barrage", "weir", "reservoir", "spillway",
    "water management", "water supply", "water treatment plant", "WTP",
    "irrigation", "irrigation canal", "canal", "water channel",
    "check dam", "anicut", "diversion weir",
    "sewerage", "sewage treatment plant", "STP",
    "wastewater treatment", "WWTP", "effluent treatment plant", "ETP",
    "drainage", "storm water drain", "surface drain",
    "pumping station", "water distribution network",
    "pipeline", "transmission main", "distribution main",
    "water storage tank", "overhead tank", "underground tank",
    
    # Airports - Infrastructure Components
    "airport", "aerodrome", "airstrip", "airfield",
    "runway", "taxiway", "apron", "tarmac",
    "terminal", "airport terminal", "passenger terminal",
    "cargo terminal", "air traffic control tower", "ATC tower",
    "hangar", "maintenance hangar",
    "instrument landing system", "ILS",
    "parking bay", "aircraft parking", "nose-in parking",
    
    # Ports & Maritime
    "port", "seaport", "harbor", "harbour", "jetty",
    "berth", "quay", "wharf", "pier",
    "container terminal", "bulk terminal", "liquid cargo terminal",
    "offshore structure", "breakwater", "sea wall",
    "dry dock", "wet dock", "ship repair facility",
    "cargo handling", "gantry crane", "ship-to-shore crane",
    "dredging", "channel deepening", "port expansion",
    
    # Buildings & Factories - Types
    "building", "structure", "construction",
    "high-rise", "high rise building", "multi-storey", "multi-storey building",
    "skyscraper", "tower", "G+", "storey",
    "commercial complex", "commercial building", "office building",
    "IT park", "IT building", "tech park",
    "residential", "residential building", "apartment building",
    "luxury residential", "premium residential",
    "villa", "luxury villa", "villa development",
    "plotted development", "row house", "townhouse",
    "industrial plant", "factory", "manufacturing facility",
    "warehouse", "logistics park", "distribution center",
    "special economic zone", "SEZ", "industrial park",
    "industrial estate", "industrial complex",
    "data center", "data centre",
    
    # Buildings - Specialized Structures
    "stadium", "sports complex", "sports arena",
    "convention center", "exhibition center",
    "shopping mall", "retail complex",
    "hospital building", "medical facility",
    "educational building", "school building", "college building",
    "hostel", "dormitory", "staff quarters",
    "hotel", "resort", "hospitality project",
    
    # Urban Infrastructure & Smart Cities
    "public infrastructure", "urban infrastructure",
    "smart city infrastructure", "smart city project",
    "urban development", "township", "integrated township",
    "new town development", "satellite city",
    "utility tunnel", "cable tunnel", "utility corridor",
    "underground utility", "services corridor",
    
    # Metro & Railways - Infrastructure
    "metro rail", "metro", "rapid transit", "mass rapid transit", "MRT",
    "rail corridor", "railway line", "track", "rail track",
    "station building", "railway station", "metro station",
    "elevated corridor", "elevated metro", "underground metro",
    "viaduct", "metro viaduct",
    "metro depot", "car shed", "stabling yard",
    "workshop", "maintenance depot", "rolling stock depot",
    "OHE", "overhead electrification", "traction power",
    "signaling", "S&T works", "signal and telecom",
    "AFC", "Automatic Fare Collection",
    "platform screen door", "PSD",
    "concourse", "platform", "mezzanine",
    "crossover", "crossover cavern", "NATM crossover",
    "cut-and-cover station", "underground station",
    "ramp", "ramp tunnel", "ventilation shaft",
    
    # Parking Structures
    "parking structure", "car park", "parking lot",
    "multilevel parking", "multi-level car parking", "MLCP",
    "underground parking", "basement parking",
    "automated parking", "mechanical parking",
    "park and ride facility",
    
    # Foundation & Structural Works
    "foundation", "deep foundation", "shallow foundation",
    "piling", "pile foundation", "bored pile", "driven pile",
    "RCC pile", "reinforced concrete pile",
    "diaphragm wall", "D-wall", "slurry wall",
    "sheet piling", "sheet pile wall",
    "raft foundation", "mat foundation",
    "caisson", "well foundation", "open caisson",
    "ground improvement", "soil stabilization",
    "stone columns", "dynamic compaction",
    
    # Materials - Concrete & Steel
    "concrete", "RCC", "reinforced concrete",
    "PCC", "plain cement concrete", "M20", "M25", "M30", "M40",
    "grade of concrete", "concrete mix design",
    "ready-mix concrete", "RMC", "batching plant",
    "precast concrete", "prestressed concrete",
    "post-tensioning", "pre-tensioning",
    "steel", "structural steel", "reinforcement", "rebar",
    "TMT bar", "thermo-mechanically treated",
    "formwork", "shuttering", "falsework",
    "scaffolding", "shoring",
    
    # Geotechnical & Site Investigation
    "geotechnical investigation", "geotechnical study",
    "soil testing", "soil investigation", "sub-soil investigation",
    "geological survey", "geo-tech report",
    "bore log", "borehole", "SPT", "standard penetration test",
    "N-value", "bearing capacity", "soil profile",
    "rock strata", "rock coring",
    
    # Project Management & Documentation
    "project management", "construction management",
    "quality control", "QC", "quality assurance", "QA",
    "safety compliance", "safety management",
    "DPR", "Detailed Project Report",
    "feasibility study", "techno-economic feasibility",
    "design document", "detailed design",
    "engineering design", "structural design",
    "construction drawing", "working drawing",
    "as-built drawing", "shop drawing",
    "bill of quantities", "BOQ", "schedule of rates", "SOR",
    
    # Contract Types & Execution
    "EPC", "Engineering Procurement Construction",
    "EPCM", "Engineering Procurement Construction Management",
    "contract", "construction contract",
    "tender", "bid", "RFP", "RFQ", "EOI", "Expression of Interest",
    "award", "LOA", "Letter of Award", "LOI", "Letter of Intent",
    "execution", "project execution", "construction execution",
    "turnkey", "turnkey project", "lump sum contract",
    "design-build", "design-bid-build",
    "item rate contract", "percentage rate contract",
    
    # MEP & Services
    "MEP", "MEP works", "MEP services",
    "mechanical", "electrical", "plumbing",
    "HVAC", "heating ventilation air conditioning",
    "air conditioning", "chiller", "AHU", "air handling unit",
    "fire fighting", "fire protection system", "sprinkler system",
    "BMS", "Building Management System",
    "electrical installation", "power distribution",
    "LT panel", "HT panel", "DG set", "diesel generator",
    "UPS", "uninterruptible power supply",
    "plumbing works", "sanitary works", "drainage works",
    
    # Finishing & Landscaping
    "finishing", "interior finishing", "external finishing",
    "flooring", "tiling", "marble flooring", "granite flooring",
    "wall finishing", "painting", "plastering",
    "false ceiling", "suspended ceiling",
    "cladding", "facade", "curtain wall", "ACP", "aluminium composite panel",
    "glazing", "glass facade", "double glazing",
    "waterproofing", "weather sealing",
    "landscaping", "horticulture", "green area development",
    "street furniture", "paving", "compound wall",
    
    # Testing & Commissioning
    "commissioning", "pre-commissioning",
    "handover", "project handover", "completion certificate",
    "material testing", "concrete testing", "cube test",
    "NDT", "non-destructive testing",
    "inspection", "quality inspection", "third-party inspection",
    "load test", "pile load test", "proof load test",
    
    # Infrastructure Authorities & Bodies
    "NHAI", "National Highways Authority of India",
    "MoRTH", "Ministry of Road Transport and Highways",
    "NHIDCL", "National Highways & Infrastructure Development Corporation",
    "DMRC", "Delhi Metro Rail Corporation",
    "MMRDA", "Mumbai Metropolitan Region Development Authority",
    "BMRCL", "Bangalore Metro Rail Corporation",
    "CMRL", "Chennai Metro Rail Limited",
    "KMRL", "Kochi Metro Rail Limited",
    "LMRC", "Lucknow Metro Rail Corporation",
    "NMRC", "Noida Metro Rail Corporation",
    "RVNL", "Rail Vikas Nigam Limited",
    "RITES", "Rail India Technical and Economic Service",
    "IRCON", "Indian Railway Construction Company",
    "AAI", "Airports Authority of India",
    "CPWD", "Central Public Works Department",
    "PWD", "Public Works Department",
    "state PWD", "municipal corporation",
    "urban development authority", "UDA",
    "development authority", "housing board",
    
    # Industrial & Specialized Projects
    "steel plant", "steel manufacturing", "blast furnace",
    "cement plant", "cement manufacturing", "kiln",
    "carbon black plant", "chemical plant",
    "refinery", "petrochemical complex",
    "power plant", "thermal power plant", "gas power plant",
    "metals and mining", "mining infrastructure",
    "ore processing", "smelter", "beneficiation plant",
    "upstream project", "downstream project",
    
    # Project Financing & Models
    "PPP", "public-private partnership",
    "BOT", "Build Operate Transfer",
    "BOOT", "Build Own Operate Transfer",
    "HAM", "Hybrid Annuity Mode",
    "EPC mode", "item rate mode",
    "infrastructure bond", "project financing",
    "infrastructure financing", "viability gap funding", "VGF",
    "annuity", "toll", "toll collection",
    
    # Standards & Compliance
    "international standards", "Indian standards", "IS code",
    "IRC", "Indian Roads Congress", "IRC specifications",
    "IS 456", "IS 1893", "IS 13920", "NBC", "National Building Code",
    "local regulations", "building bylaws", "zoning regulations",
    "environmental clearance", "EC", "forest clearance",
    "CRZ", "Coastal Regulation Zone", "CRZ clearance",
    "social impact assessment", "SIA",
    "rehabilitation", "R&R", "resettlement",
    "land acquisition", "right of way", "RoW",
    
    # Project Metrics & Timeline
    "project timeline", "construction schedule",
    "milestone", "completion", "COD", "completion date",
    "SCOD", "Scheduled Completion Date",
    "delay", "time extension", "liquidated damages", "LD",
    "penalty", "bonus clause",
    "progress report", "physical progress", "financial progress",
    "project monitoring", "progress monitoring",
    
    # Safety, Health & Environment
    "safety", "health", "environment",
    "SHE", "SHEQ", "HSE", "EHS",
    "safety protocol", "safety audit", "safety officer",
    "PPE", "personal protective equipment",
    "accident", "near miss", "safety incident",
    "environmental impact assessment", "EIA",
    "environmental management plan", "EMP",
    "pollution control", "dust suppression", "noise control",
    
    # Workforce & Equipment
    "skilled labor", "skilled labour", "workforce",
    "manpower", "labor deployment", "labour deployment",
    "construction machinery", "heavy equipment",
    "equipment rental", "plant and machinery",
    "excavator", "bulldozer", "loader", "backhoe",
    "crane", "tower crane", "mobile crane",
    "concrete mixer", "transit mixer", "batching plant",
    "paver", "paving machine", "roller", "compactor",
    "grader", "motor grader", "earth moving equipment",
    
    # Real Estate & Developers
    "real estate", "real estate developer", "property developer",
    "builder", "construction company",
    "marquee client", "prestigious client", "reputed developer",
    "western India", "northern India", "southern India", "eastern India",
    "square feet", "sq ft", "lakh square feet", "built-up area",
    "carpet area", "super built-up area",
    
    # Buildings & Factories Segment (KEC-specific)
    "Buildings & Factories", "B&F segment",
    "repeat order", "repeat client",
    "YTD order intake", "year-to-date",
    "order book", "civil order book",
    
    # Additional Technical Terms
    "retaining wall", "RE wall", "breast wall",
    "culvert", "box culvert", "slab culvert", "pipe culvert",
    "drainage structure", "cross drainage work",
    "gabion wall", "mechanically stabilized earth", "MSE wall",
    "soil nailing", "slope protection",
    "dewatering", "wellpoint system",
    "temporary works", "construction methodology",
    "lifting plan", "erection sequence",
    "pre-stressing", "launching", "launching girder",
    "incremental launching", "balanced cantilever"
    ],

    "Transportation": [
      # Core Railway & Transit Terms
    "railway", "rail", "railroad", "train", "locomotive",
    "metro", "rapid transit", "mass rapid transit", "MRT",
    "metro rail", "metro railway", "urban transit", "urban railway",
    "metro system", "metro network", "metro corridor", "metro project",
    
    # Metro Types & Systems
    "metro line", "metro phase", "metro phase 1", "metro phase 2", "metro phase 3",
    "metro extension", "metro expansion", "elevated metro", "underground metro",
    "hybrid metro", "mixed metro", "at-grade metro",
    
    # Alternative Rail Systems
    "monorail", "light rail", "light rapid transit", "LRT",
    "elevated corridor", "elevated viaduct", "metro viaduct",
    "people mover", "automated people mover", "APM",
    
    # High-Speed & Regional Rail (KEC-Specific)
    "high-speed rail", "HSR", "bullet train", "speed corridor",
    "semi-high-speed rail", "semi-HSR", "higher speed rail",
    "regional rapid transit", "RRTS", "rapid rail", "RapidX",
    "RRTS corridor", "RRTS phase",
    "design speed", "operational speed", "maximum speed",
    "180 km/h", "160 km/h", "200 km/h", "300 km/h", "kmph",
    
    # Project-Specific Terminology
    "Delhi-Meerut RRTS", "Delhi-Meerut corridor",
    "NCRTC", "National Capital Region Transport Corporation",
    "Namo Bharat", "Delhi-Alwar RRTS", "Delhi-Panipat RRTS",
    "Mumbai-Ahmedabad HSR", "MAHSR", "bullet train project",
    "NHSRCL", "National High Speed Rail Corporation Limited",
    "semi-high-speed", "160-200 km/h", "speed range",
    
    # Metro Authorities & Operators (India)
    "DMRC", "Delhi Metro Rail Corporation",
    "MMRC", "Mumbai Metro Rail Corporation", "MMRDA",
    "KMRL", "Kochi Metro Rail Limited",
    "CMRL", "Chennai Metro Rail Limited",
    "BMRCL", "Bangalore Metro Rail Corporation",
    "LMRC", "Lucknow Metro Rail Corporation",
    "NMRC", "Noida Metro Rail Corporation",
    "GMRC", "Gujarat Metro Rail Corporation",
    "Jaipur Metro", "Hyderabad Metro Rail", "HMRL",
    "Mumbai Metro One", "MMOPL",
    
    # Railway Authorities
    "Indian Railways", "IR", "Ministry of Railways",
    "RVNL", "Rail Vikas Nigam Limited",
    "RITES", "Rail India Technical and Economic Service",
    "IRCON", "Indian Railway Construction Company",
    "IRSDC", "Indian Railway Station Development Corporation",
    "RDSO", "Research Design & Standards Organisation",
    "CRS", "Chief Commissioner of Railway Safety",
    
    # Metro Project Components (KEC-Specific Work)
    "station", "metro station", "RRTS station",
    "viaduct", "elevated viaduct", "double-decker viaduct",
    "viaduct construction", "viaduct bridge",
    "depot", "metro depot", "depot-cum-workshop",
    "car shed", "carriage maintenance depot",
    "workshop", "maintenance workshop", "workshop facility",
    
    # Track & Infrastructure
    "track", "rail track", "permanent way", "PW",
    "sleeper", "railway sleeper", "concrete sleeper", "steel sleeper",
    "ballast", "ballasted track", "ballastless track",
    "rail gauge", "standard gauge", "broad gauge", "meter gauge",
    "1676 mm", "1600 mm", "1435 mm",
    "rail jointless track", "continuous welded rail", "CWR",
    "turnout", "points", "crossing",
    
    # Station Infrastructure
    "platform", "island platform", "side platform",
    "platform screen door", "PSD", "safety door",
    "waiting area", "concourse", "station concourse",
    "entrance", "exit", "entry/exit",
    "ticket counter", "booking office",
    "passenger amenities", "passenger facilities",
    "accessibility", "disabled access", "universal design",
    "toilet", "restroom", "drinking water facility",
    "public address system", "PA system",
    
    # Train Control & Operations
    "signaling", "signalling", "train control", "traffic control",
    "ATC", "Automatic Train Control", "ATP", "Automatic Train Protection",
    "Kavach", "TCAS", "Train Collision Avoidance System",
    "ETCS", "European Train Control System",
    "cab signaling", "in-cab signalling", "CBTC", "Communication Based Train Control",
    "CATC", "Continuous Automatic Train Control",
    "electronic interlocking", "EI", "route relay interlocking", "RRI",
    "signal", "lineside signal", "approach signal",
    "block section", "absolute block", "automatic block",
    "movement authority", "speed restriction", "temporary speed restriction", "TSR",
    "GSM-R", "LTE-R", "radio communication",
    "RFID", "Radio Frequency Identification", "RFID tags",
    "level crossing", "grade crossing", "automatic level crossing",
    "grade separation", "grade separated crossing",
    "interlocking", "interlocked", "non-interlocked",
    "network monitoring system", "NMS",
    "traffic management system", "TMS",
    
    # Ticketing & Fare Collection
    "ticketing", "ticket system", "integrated ticketing",
    "fare collection", "AFC", "Automatic Fare Collection",
    "smart card", "contactless card", "open-loop system",
    "RFID", "near-field communication", "NFC",
    "payment gateway", "mobile payment", "QR code",
    "POM", "Proof of Payment", "ticket validator",
    "revenue management", "revenue collection",
    
    # Station Design & Architecture
    "station design", "architectural design", "station architecture",
    "aesthetic planning", "aesthetic design",
    "beautiful station", "world-class station",
    "station typology", "station category",
    "grade-separated station", "at-grade station",
    "station building", "station structure",
    
    # Vertical Circulation & MEP
    "escalator", "elevator", "lift",
    "stairs", "staircase", "emergency stairs",
    "emergency exit", "evacuation route",
    "ventilation system", "HVAC", "air conditioning",
    "cooling system", "heating system",
    "fire safety system", "fire suppression",
    "emergency lighting", "backup power",
    "water supply system", "drainage system",
    "sewerage system", "waste management",
    "electrical system", "power distribution",
    "traction power supply", "auxiliary power",
    
    # Rolling Stock & Trains
    "rolling stock", "train set", "trainset",
    "coach", "carriage", "car", "passenger coach",
    "motor car", "trailer car", "driving motor car", "DMC",
    "articulated coach", "semi-permanent coupler",
    "bogie", "wheel set", "axle", "wheel",
    "suspension system", "shock absorber", "springs",
    "coupler", "automatic coupler", "semi-permanent coupler",
    "cabin", "driver cabin", "driver cab",
    "passenger seating", "seat", "grab handle", "handrail",
    "door", "sliding door", "automatic door",
    "interior design", "cabin design", "ergonomic design",
    "light weight design", "low-floor train",
    "air conditioning", "passenger comfort",
    "VVVF", "Variable Voltage Variable Frequency",
    "regenerative braking", "braking system", "wheel disc brake",
    
    # Traction System
    "traction", "traction power", "power supply system",
    "electric traction", "AC traction", "DC traction",
    "25 kV AC", "25kV AC", "750V DC", "1500V DC",
    "overhead catenary system", "OCS", "overhead line equipment", "OHE",
    "catenary", "contact line", "OCL", "overhead contact line",
    "pantograph", "current collector",
    "third rail system", "conductor rail", "third rail",
    "power supply feeder", "substation", "traction substation",
    "neutral section", "neutral zone", "insulated overlap",
    "feeding section", "booster transformer",
    "traction rectifier", "converter", "inverter",
    "3-phase induction motor", "traction motor",
    "gearbox", "gear ratio", "motor drive",
    "regenerative braking", "braking energy recovery",
    
    # Depot & Workshop
    "stabling", "rake stabling", "train stabling",
    "stabling line", "stabling facility",
    "inspection", "inspection line", "inspection shed",
    "maintenance", "scheduled maintenance", "periodic maintenance",
    "servicing", "overhaul", "major overhaul", "MOH",
    "periodical overhaul", "POH",
    "unscheduled maintenance", "breakdown maintenance",
    "wheel profiling", "wheel lathe",
    "paint shop", "interior cleaning", "exterior cleaning",
    "coach washing", "automatic coach washing", "coach wash plant",
    "underframe cleaning", "roof cleaning",
    "repair workshop", "electrical workshop",
    "store depot", "spare parts storage",
    "tools & equipment storage", "test equipment",
    "Operation Control Centre", "OCC", "administrative building",
    "CCTV surveillance", "24/7 monitoring",
    
    # Maintenance & Operations
    "preventive maintenance", "predictive maintenance",
    "condition monitoring", "online monitoring", "health monitoring",
    "diagnostics", "fault diagnostics", "fault analysis",
    "reliability", "availability", "maintainability", "RAM",
    "mean time between failures", "MTBF",
    "mean time to repair", "MTTR",
    "operational efficiency", "service quality", "punctuality",
    "frequency", "operational frequency", "headway",
    "capacity", "train capacity", "seating capacity", "crush load",
    "ridership", "passenger traffic", "occupancy",
    
    # Tunnel & Underground Construction
    "tunnel", "metro tunnel", "railway tunnel", "underground tunnel",
    "tunnel section", "tunnel boring", "tunneling", "tunnelling",
    "TBM", "Tunnel Boring Machine", "EPB", "Earth Pressure Balance",
    "NATM", "New Austrian Tunneling Method", "New Austrian Tunnelling Method",
    "cut-and-cover", "cut and cover", "cover-and-cut", "top-down method",
    "drill and blasting", "D&B", "drilling and blasting",
    "shield tunneling", "shield machine", "open shield",
    "shotcrete", "rock bolting", "rock support",
    "lattice girder", "steel support frame",
    "temporary support", "permanent lining",
    "primary lining", "secondary lining", "temporary lining",
    "waterproofing", "grouting", "injection grouting",
    "portal", "tunnel portal", "approach tunnel",
    "ventilation shaft", "escape shaft", "cross passage", "cross-passage",
    "cavern", "station cavern", "NATM crossover cavern",
    "invert", "tunnel invert", "base slab",
    "piling", "pipe piling", "secant piling", "tangent piling",
    "diaphragm wall", "D-wall", "slurry wall",
    "dewatering", "wellpoint dewatering", "groundwater management",
    
    # Bridge & Viaduct Construction
    "bridge", "railway bridge", "metro bridge",
    "viaduct", "elevated structure", "elevated guideway",
    "span", "bridge span", "viaduct span",
    "pier", "column", "pylon", "abutment",
    "pre-stressed concrete", "PSC", "precast concrete",
    "box girder", "I-girder", "steel girder",
    "concrete casting", "casting yard", "batch plant",
    "launching", "incremental launching", "balanced cantilever",
    "Full Span Launching Method", "FSLM",
    "girder erection", "equipment placement",
    "bearing", "expansion joint", "contraction joint",
    "parapet", "crash barrier", "safety barrier",
    "waterproofing", "drainage system",
    "decking", "wear coat", "surface finish",
    
    # Alignment & Route Planning
    "corridor", "rail corridor", "transport corridor",
    "route", "alignment", "route alignment",
    "level", "grade", "gradient", "slope",
    "horizontal curve", "vertical curve", "curve radius",
    "grade separation", "grade-separated", "at-grade",
    "elevated section", "underground section", "at-grade section",
    "terminal", "end-of-line terminal", "intermediate terminal",
    "inter-modal connectivity", "intermodal integration",
    "feeder service", "feeder bus", "first-last mile",
    "park and ride", "multi-modal transport hub",
    
    # Project Planning & Design
    "DPR", "Detailed Project Report",
    "feasibility study", "techno-economic feasibility",
    "master plan", "development plan",
    "engineering drawing", "technical drawing",
    "single line diagram", "SLD", "general arrangement", "GA",
    "design specifications", "technical specifications",
    "working drawing", "construction drawing",
    "shop drawing", "as-built drawing", "record drawing",
    "bill of quantities", "BOQ", "schedule of rates", "SOR",
    "project schedule", "timeline", "Gantt chart", "critical path",
    "milestone", "major milestone", "phase milestone",
    
    # Project Execution & Commissioning
    "EPC", "Engineering Procurement Construction",
    "contract", "construction contract", "turnkey contract",
    "tender", "bid", "competitive bid", "international tender",
    "RFP", "Request for Proposal", "RFQ", "Request for Quotation",
    "award", "LOA", "Letter of Award", "LOI", "Letter of Intent",
    "execution", "project execution", "construction execution",
    "commissioning", "testing and commissioning", "T&C",
    "pre-commissioning test", "PRCR", "Physical Run Capability Record",
    "trial run", "test run", "operational trial",
    "handover", "operational handover", "operational readiness",
    "COD", "Completion Date", "SCOD", "Scheduled Completion Date",
    "completion certificate", "defect liability period", "DLP",
    "performance guarantee", "liquidated damages", "LD", "delay charges",
    
    # Civil Works (Metro/Rail Specific)
    "civil works", "civil construction",
    "structural works", "concrete works", "steel works",
    "excavation", "excavation works", "earth moving",
    "site preparation", "site clearing",
    "foundation", "deep foundation", "foundation piling",
    "bored pile", "large diameter pile", "LDP",
    "reinforced concrete", "RCC", "concrete grade",
    "formwork", "shuttering", "falsework",
    "scaffolding", "temporary support system",
    "quality control", "QC", "quality assurance", "QA",
    "material testing", "cube test", "slump test", "compression test",
    "NDT", "non-destructive testing",
    "inspection", "site inspection", "third-party inspection",
    "load test", "proof load test", "load bearing test",
    
    # MEP Works for Stations & Depots
    "MEP", "Mechanical Electrical Plumbing",
    "mechanical works", "electrical works", "plumbing works",
    "HVAC", "heating ventilation air conditioning",
    "air conditioning system", "chiller plant",
    "firefighting system", "fire suppression system", "sprinkler system",
    "fire detection", "fire alarm system",
    "electrical installation", "power distribution",
    "low-tension panel", "LT panel", "high-tension panel", "HT panel",
    "lighting system", "emergency lighting", "exit lighting",
    "DG set", "diesel generator", "backup power",
    "UPS", "uninterruptible power supply",
    "plumbing system", "water supply", "hot water supply",
    "drainage system", "sewage treatment", "wastewater management",
    "BMS", "Building Management System", "automation system",
    "security system", "access control",
    
    # Systems Integration
    "systems integration", "integrated system",
    "ticketing system integration", "fare system",
    "information system", "passenger information system", "PIS",
    "CCTV system", "surveillance system", "security CCTV",
    "PA system", "public address system", "announcements",
    "clock system", "synchronized clock system",
    "control system", "train control system",
    "SCADA", "Supervisory Control and Data Acquisition",
    "EMS", "Energy Management System",
    "wireless communication", "Wi-Fi", "mobile connectivity",
    
    # OEM & Supply
    "OEM", "Original Equipment Manufacturer",
    "rolling stock supplier", "train manufacturer",
    "equipment supplier", "subsystem supplier",
    "technology provider", "software provider",
    "signaling system supplier", "track provider",
    "cable supplier", "electrical equipment supplier",
    
    # Regional Markets & Cities
    "pan-India", "all-India", "national level",
    "region-specific", "regional metro", "city metro",
    "state capital", "national capital", "metropolitan city",
    "tier-I city", "tier-II city", "tier-III city",
    "major cities", "urban agglomeration",
    
    # Metro Cities (KEC-Specific Projects)
    "Delhi", "Delhi Metro", "DMRC", "Delhi NCR",
    "Kochi", "Kochi Metro", "KMRL",
    "Chennai", "Chennai Metro", "CMRL",
    "Bangalore", "Bangalore Metro", "BMRCL", "Namma Metro",
    "Mumbai", "Mumbai Metro", "MMRC",
    "Lucknow", "Lucknow Metro", "LMRC",
    "Noida", "Noida Metro", "NMRC",
    "Jaipur", "Jaipur Metro",
    "Hyderabad", "Hyderabad Metro", "HMRL",
    "Pune", "Pune Metro",
    "Ahmedabad", "Ahmedabad Metro",
    "Kolkata", "Kolkata Metro",
    "Chennai", "Kochi", "Thiruvananthapuram",
    "Pune", "Visakhapatnam", "Bhopal", "Nagpur",
    
    # RRTS/HSR Routes
    "Delhi-Meerut", "Delhi-Alwar", "Delhi-Panipat",
    "Mumbai-Ahmedabad", "bullet train corridor",
    "Delhi-Varanasi", "Delhi-Ahmedabad HSR",
    "Mumbai-Nagpur", "Mumbai-Hyderabad",
    "Chennai-Mysore", "Delhi-Amritsar",
    
    # Government Initiatives
    "NITI Aayog", "Ministry of Railways",
    "urban development ministry",
    "National Transit System", "national master plan",
    "metro mission", "urban transport mission",
    "smart cities mission", "smart city project",
    "sustainable transport", "low-carbon transport",
    "integrated transport", "multi-modal transport",
    
    # Operational Metrics & KPIs
    "frequency", "train frequency", "headway",
    "ridership", "daily ridership", "annual ridership",
    "capacity", "line capacity", "system capacity",
    "occupancy rate", "load factor",
    "demand forecast", "traffic demand",
    "CAGR", "compound annual growth rate", "growth rate",
    "revenue", "fare revenue", "commercial revenue",
    "operational efficiency", "cost per km", "operating cost",
    "maintenance cost", "life cycle cost",
    "asset management", "asset utilization",
    "safety record", "incident rate", "accident rate",
    "punctuality", "on-time performance", "service reliability",
    "customer satisfaction", "service quality",
    "environmental impact", "carbon emissions reduction",
    "noise level", "vibration", "noise pollution",
    
    # Standards & Compliance
    "international standards", "ISO", "Indian standards",
    "IRC", "Indian Roads Congress", "Indian Railway Standards",
    "IEEE standards", "IEC standards",
    "safety standards", "fire safety standards",
    "environmental standards", "pollution control standards",
    "local norms", "grid codes", "building bylaws",
    "accessibility standards", "universal design standards",
    "Indian Railway Board", "CRS clearance",
    "safety certification", "technical clearance",
    
    # Project Challenges & Management
    "project delay", "time overrun", "schedule delay",
    "cost overrun", "budget overrun", "financial closure",
    "change order", "variation order", "VO",
    "delay liquidated damages", "LD charges",
    "completion deadline", "project timeline",
    "project risk", "technical risk", "financial risk",
    "delay reasons", "time extension", "force majeure",
    "project monitoring", "progress report", "performance monitoring",
    "independent engineer", "IE", "technical auditor",
    
    # Safety, Health & Environment
    "safety", "health", "environment",
    "SHE", "SHEQ", "HSE", "EHS",
    "safety protocol", "safety audit", "safety audit report",
    "accident prevention", "accident statistics", "incident reporting",
    "PPE", "personal protective equipment",
    "near miss", "safety incident", "accident investigation",
    "environmental impact assessment", "EIA", "environmental clearance", "EC",
    "environmental management plan", "EMP",
    "social impact assessment", "SIA",
    "noise pollution", "air pollution", "dust control",
    "vibration monitoring", "environmental monitoring",
    "rehabilitation", "resettlement", "R&R",
    "land acquisition", "land requirement",
    "community engagement", "public consultation",
    
    # Workforce & Training
    "skilled manpower", "skilled labour", "skilled labor",
    "workforce", "workforce deployment",
    "labor deployment", "labour deployment",
    "training", "capacity building", "skill development",
    "staff training", "operator training", "maintenance training",
    "multi-skilling", "cross-functional training",
    "operational staff", "maintenance staff", "administrative staff",
    "recruitment", "recruitment plan",
    
    # Financial & Investment
    "capex", "capital expenditure", "investment",
    "opex", "operational expenditure", "operating cost",
    "project cost", "total project cost", "TPC",
    "budget", "budget allocation", "budget management",
    "cost estimation", "cost optimization",
    "financing", "project financing", "infrastructure financing",
    "PPP", "public-private partnership", "government funding",
    "viability gap funding", "VGF", "subsidy",
    "annuity model", "revenue sharing", "toll-based",
    
    # Order & Business Metrics
    "order intake", "order book", "YTD", "year-to-date",
    "order value", "contract value", "deal size",
    "repeat order", "extension order", "new order",
    "order announcement", "press release",
    "client acquisition", "marquee client", "prestigious client",
    "customer diversification", "geographic diversification",
    "market presence", "footprint", "country presence",
    "revenue contribution", "revenue visibility",
    
    # Competitive & Strategic
    "L&T", "Larsen & Toubro", "Tata Projects", "Afcons",
    "NPCC", "national contractors",
    "Chinese contractors", "international contractors",
    "competitive advantage", "market position",
    "first order", "maiden order", "landmark project",
    "repeat business", "long-term relationship",
    "consortium", "joint venture", "partnership",
    "subcontractor", "sub-supply", "supply chain"  ],

    "Oil & Gas": [    # Core Oil & Gas Terminology
    "oil", "gas", "crude oil", "crude", "petroleum",
    "natural gas", "LNG", "liquefied natural gas",
    "condensate", "NGL", "natural gas liquids",
    "hydrocarbon", "hydrocarbons",
    
    # Value Chain Segments
    "upstream", "midstream", "downstream",
    "exploration", "production", "expl-prod", "E&P",
    "drilling", "drilling operation", "drilling rig",
    "well", "borehole", "wellhead", "well site",
    "production facility", "production platform",
    "extraction", "recovery", "crude recovery",
    "reserves", "resource", "resource development",
    
    # Offshore Infrastructure - Location Types
    "onshore", "onshore pipeline", "onshore facility",
    "offshore", "offshore pipeline", "offshore field",
    "shallow water", "shallow shelf",
    "deepwater", "deep-water field", "ultra-deepwater",
    "ultra-deepwater", "UDW", "subsea field",
    "shelf", "continental shelf",
    "seabed", "sea floor", "seafloor",
    
    # Offshore Platforms & Structures
    "platform", "offshore platform", "production platform",
    "fixed platform", "jacket platform", "steel platform",
    "floating platform", "spar platform", "tension leg platform", "TLP",
    "FPSO", "Floating Production Storage Offloading",
    "FSO", "Floating Storage Offloading",
    "FDS", "Field Development Ship",
    "flotel", "floating hotel",
    "mooring system", "spread mooring", "turret mooring",
    "single point mooring", "SPM",
    "manifold", "subsea manifold", "production manifold",
    "template", "subsea template", "wellhead template",
    "tree", "subsea tree", "mudline", "wet-tree",
    
    # Subsea Systems - SURF (Subsea Umbilicals, Risers, Flowlines)
    "SURF", "Subsea Umbilicals Risers Flowlines",
    "subsea system", "subsea infrastructure",
    "subsea", "underwater", "sub-sea",
    
    
    # Flowlines
    "flowline", "flowlines", "production flowline",
    "export line", "export flowline", "export pipeline",
    "import line", "water injection line", "gas injection line",
    "feeder line", "tributary line",
    "subsea flowline", "underwater flowline",
    "pipeline diameter", "line size",
    "12 inch", "14 inch", "16 inch", "18 inch", "20 inch",
    "24 inch", "30 inch", "36 inch", "42 inch",
    "pipeline wall thickness", "heavy wall", "sour service",
    
    # Pipeline Infrastructure
    "pipeline", "pipelines", "pipeline system",
    "pipeline project", "pipeline construction",
    "trunk line", "main pipeline", "main transmission",
    "feeder line", "gathering line", "collection line",
    "tie-in", "pipeline tie-in", "interconnection",
    "onshore pipeline", "offshore pipeline",
    "land pipeline", "subsea pipeline", "underwater pipeline",
    "cross-country pipeline", "long-distance pipeline",
    "interstate pipeline", "inter-state transmission",
    "coast-to-coast", "landfall", "offshore-to-onshore",
    
    # Pipeline Specifications & Operations
    "pressure", "PSI", "bar", "MPa",
    "MOPQ", "MAOP", "Maximum Operating Pressure",
    "design pressure", "working pressure",
    "high-pressure pipeline", "low-pressure pipeline",
    "pressure rating", "pressure class",
    "flow rate", "throughput", "capacity",
    "MMSCFD", "million standard cubic feet per day",
    "MMSCM", "million standard cubic meters",
    "BPSD", "barrels per stream day",
    "linepack", "surge capacity", "storage capacity",
    "line rupture", "integrity", "safe operation",
    
    # Pipeline Protection & Materials
    "corrosion protection", "corrosion resistance",
    "coating", "pipeline coating", "epoxy coating",
    "fusion bonded epoxy", "FBE", "polyethylene coating", "PE",
    "cathodic protection", "CP", "sacrificial anode",
    "impressed current system", "ICCS",
    "material", "carbon steel", "stainless steel", "duplex steel",
    "martensitic steel", "nickel alloy",
    "heavy wall", "sour service", "sour gas",
    "H2S resistant", "CO2 resistant", "corrosive service",
    "tensile strength", "yield strength", "X42", "X52", "X60", "X70", "X80",
    "metallurgy", "material selection", "material certification",
    
    # Pipeline Engineering & Design
    "pipeline layout", "route survey", "survey", "bathymetry",
    "engineering", "design", "detailed design",
    "specification", "specifications", "spec sheet",
    "pipe specification", "equipment specification",
    "technical drawings", "engineering drawings",
    "3D model", "CAD", "piping & instrumentation diagram", "P&ID",
    "flow assurance", "thermodynamic analysis",
    "slug catcher", "scraper trap", "trap",
    "pig launcher", "pig receiver", "intelligent pigging",
    
    # Pipeline Installation Methods
    "installation", "laying", "pipe laying",
    "J-lay", "S-lay", "reel lay", "tow-in method",
    "J-lay vessel", "S-lay vessel", "pipelay barge",
    "crane barge", "heavy-lift vessel", "installation vessel",
    "DP vessel", "dynamic positioning", "DP ship",
    "anchor handling tug supply", "AHTS",
    "pipe string", "pipe section", "joint",
    "welding", "field weld", "on-bottom", "stinger",
    "backfill", "burial", "trenching", "seabed intervention",
    
    # Pipeline Components & Equipment
    "pipe mill", "mill test certificate", "MTC",
    "pipe supplier", "fabrication", "prefab",
    "valve", "isolation valve", "block valve",
    "check valve", "pressure relief valve", "PRV",
    "butterfly valve", "ball valve", "gate valve",
    "fitting", "coupling", "connector",
    "flange", "flanged connection", "welded connection",
    "reducer", "tee", "elbow", "bend",
    "strainer", "filter", "scraper trap",
    "expansion loop", "anchor block", "support",
    "clamp", "cable tray", "marker buoy",
    
    # Pipeline Testing & Commissioning
    "commissioning", "testing", "test & commissioning", "T&C",
    "pressure test", "hydrostatic test", "strength test",
    "integrity", "strength integrity", "operational integrity",
    "inspection", "pre-commissioning inspection", "PCI",
    "NDT", "non-destructive testing", "ultrasonic testing", "UT",
    "radiography", "X-ray", "magnetic particle inspection", "MPI",
    "dye penetrant", "DP test", "visual inspection", "VI",
    "commissioning process", "line fill", "line displacement",
    "pigging", "gauging pig", "smart pig", "inspection pig",
    "batching", "batch operation", "batch separation",
    "pre-operation", "handover", "operational readiness",
    
    # Subsea Installation & Techniques
    "subsea installation", "underwater installation",
    "cable laying", "cable burial", "cable trenching",
    "heavy lift", "heavy-lift operation", "heavy-lift vessel",
    "module installation", "topsides installation",
    "hook-up", "hook-up and commissioning", "H&C",
    "pre-installation", "infield installation",
    "tie-in", "subsea tie-in", "connection",
    "marine operation", "offshore operation",
    "vessel mobilization", "mobilization", "demobilization",
    "weather window", "sea state", "environmental conditions",
    
    # Refinery Terms & Infrastructure
    "refinery", "oil refinery", "refining facility",
    "greenfield refinery", "brownfield refinery", "expansion refinery",
    "refining capacity", "refining throughput",
    "barrels per day", "BPD", "barrel per stream day", "BPSD",
    "MMTPA", "Million Metric Tonnes Per Annum",
    "KTPA", "Thousand Tonnes Per Annum",
    "capacity", "effective capacity", "nameplate capacity",
    "utilization", "capacity utilization", "run rate",
    "process unit", "processing unit", "refinery unit",
    
    # Refinery Processes - Primary
    "crude distillation", "atmospheric distillation",
    "crude distillation unit", "CDU",
    "vacuum distillation", "vacuum unit", "VDU",
    "fractionation", "fractionator", "distillation column",
    "feedstock", "crude blend", "mixed crude",
    "light crude", "heavy crude", "sour crude", "sweet crude",
    
    # Refinery Processes - Secondary Processing
    "cracking", "catalytic cracking", "FCC", "Fluid Catalytic Cracking",
    "hydrocracking", "hydrocracker", "HC unit",
    "reforming", "catalytic reforming", "naphtha reforming",
    "reformer", "regenerator", "reactor",
    "delayed coking", "coking unit", "coke",
    "isomerization", "isomerizer", "ISOM",
    "alkylation", "alkylate", "polymer",
    "polymerization", "polymerizer",
    
    # Refinery Products & Yields
    "product", "refinery product", "petroleum product",
    "gasoline", "MS", "motor spirit",
    "diesel", "high-speed diesel", "HSD",
    "ATF", "aviation turbine fuel", "jet fuel",
    "fuel oil", "LSHS", "low sulphur heavy stock",
    "LPG", "liquefied petroleum gas", "propane", "butane",
    "naphtha", "light naphtha", "heavy naphtha",
    "keroset", "kerosene",
    "aromatics", "BTX", "benzene", "toluene", "xylene",
    "base oil", "lube base oil", "LOBS", "lube oil base stock",
    "bitumen", "asphalt", "bituminous material",
    "hydrogen", "fuel gas", "sulfur", "coke",
    "product yield", "yield curve",
    
    # Product Quality Parameters
    "octane rating", "research octane number", "RON",
    "motor octane number", "MON", "antiknock index", "AKI",
    "sulphur content", "sulphur level", "total sulphur",
    "cetane number", "cetane index", "CI",
    "pour point", "flashpoint", "viscosity",
    "API gravity", "gravity", "specific gravity",
    "Reid vapor pressure", "RVP",
    "aromatic content", "olefin content", "saturation",
    "distillation", "ASTM distillation", "D86",
    "water content", "sediment", "trace metals",
    "environmental compliance", "specifications",
    
    # Petrochemical Complexes & Products
    "petrochemical", "petrochemicals", "petrochem complex",
    "integrated complex", "refining petrochemical",
    "ethylene", "ethylene production", "ethylene cracker",
    "propylene", "propane dehydrogenation", "PDH",
    "BTX", "aromatics extraction", "aromatic unit",
    "benzene", "toluene", "xylene", "mixed xylene",
    "cracker", "naphtha cracker", "steam cracker",
    "olefin", "monomer", "polymer",
    "polyethylene", "LDPE", "LLDPE", "HDPE",
    "polypropylene", "PP", "polystyrene",
    "PET", "polyethylene terephthalate",
    "specialty chemical", "specialty polymer",
    "adhesive", "resin", "coating", "paint",
    
    # Petrochemical Feedstocks & Chemicals
    "feedstock", "cracker feedstock", "chemical feedstock",
    "syngas", "synthesis gas", "CO + H2",
    "methanol", "methanol synthesis",
    "ammonia", "ammonia synthesis", "urea",
    "hydrogen", "hydrogen production", "steam reforming",
    "carbon monoxide", "CO", "CO2", "carbon dioxide",
    "glycol", "ethylene glycol", "propylene glycol",
    "acetic acid", "acetic anhydride",
    "formaldehyde", "phenol", "acetone",
    
    # Polymer & Plastic Products
    "polymer", "polymeric", "plastic", "plastic resin",
    "polymerization", "polymerization reactor",
    "catalyst", "catalyst system", "metallocene",
    "additive", "stabilizer", "antioxidant", "UV stabilizer",
    "plasticizer", "filler", "reinforcement",
    "processing", "extrusion", "injection molding",
    "film", "fiber", "sheet", "pellet",
    "blow molding", "thermoforming", "3D printing",
    
    # Industrial Chemicals
    "fertilizer", "nitrogenous fertilizer",
    "urea", "ammonia nitrate", "ammonium sulphate",
    "phosphate", "phosphoric acid",
    "potash", "K2O", "potassium chloride",
    "sodium chlor-alkali", "chlor-alkali",
    "caustic soda", "chlorine", "HCl",
    "sulfuric acid", "oleum",
    "nitric acid", "phosphorous acid",
    
    # Project Development Stages
    "project development", "field development",
    "development phase", "project phase",
    "front-end loading", "FEL", "FEL-0", "FEL-1", "FEL-2", "FEL-3",
    "concept", "pre-FEL", "concept study", "concept selection",
    "basis of design", "BOD",
    "DPR", "Detailed Project Report",
    "FEED", "Front-End Engineering Design",
    "basic engineering", "BE", "basic design",
    "detailed engineering", "DE", "detail design",
    "engineering", "detailed engineering design",
    "engineering drawing", "engineering specification",
    "technical drawing", "construction drawing",
    "P&ID", "Piping & Instrumentation Diagram",
    "equipment schedule", "equipment list",
    
    # Project Execution Models
    "EPC", "Engineering Procurement Construction",
    "EPCI", "Engineering Procurement Construction Installation",
    "EPIC", "Engineering Procurement Installation Construction",
    "EPCM", "Engineering Procurement Construction Management",
    "turnkey", "turnkey contract", "turnkey project",
    "lump-sum", "lump-sum contract",
    "reimbursable", "cost-plus", "cost-plus contract",
    "target cost", "target price",
    "fixed price", "fixed cost", "fixed-price contract",
    "open-ended", "open-ended contract",
    "time & material", "T&M",
    "unit price", "rate contract",
    
    # Contracting & Bidding
    "contract", "construction contract",
    "tender", "tender process", "tendering",
    "bid", "bidding", "competitive bid",
    "RFQ", "Request for Quotation",
    "RFP", "Request for Proposal",
    "EOI", "Expression of Interest",
    "proposal", "bid proposal", "technical bid",
    "financial bid", "price bid",
    "award", "contract award",
    "LOA", "Letter of Award",
    "LOI", "Letter of Intent",
    "contract signing", "mobilization",
    
    # Project Execution & Delivery
    "execution", "project execution", "implementation",
    "mobilization", "mobilization phase",
    "demobilization", "demob",
    "engineering phase", "procurement phase", "construction phase",
    "site work", "field work", "on-site activity",
    "schedule", "project schedule", "execution schedule",
    "critical path", "critical path method", "CPM",
    "milestone", "major milestone", "key milestone",
    "completion", "project completion",
    "completion date", "scheduled completion date",
    "final handover", "operational handover",
    "warranty period", "performance guarantee",
    
    # Quality & Compliance
    "quality management", "quality assurance", "QA",
    "quality control", "QC", "inspection",
    "documentation", "technical documentation",
    "material certification", "certificate of conformity",
    "third party inspection", "TPI", "third party validator",
    "compliance", "regulatory compliance",
    "code of practice", "international standards",
    "API", "American Petroleum Institute",
    "ASME", "American Society of Mechanical Engineers",
    "DNV", "Det Norske Veritas", "classification society",
    "ABS", "American Bureau of Shipping",
    "Lloyd", "Lloyd's Register", "classification",
    "ISO", "International Organization Standardization",
    "industry standards", "industry practice",
    
    # Project Management & Planning
    "project management", "project planning",
    "cost estimation", "cost estimate", "cost planning",
    "budget", "project budget", "budgeting",
    "contingency", "contingency reserve", "reserve",
    "risk management", "risk assessment", "risk mitigation",
    "project risk", "schedule risk", "cost risk", "technical risk",
    "change management", "change order", "variation order",
    "cost overrun", "schedule overrun", "time overrun",
    "delay", "project delay", "schedule delay",
    "dispute resolution", "arbitration",
    
    # Safety, Health & Environment
    "HSE", "health safety environment",
    "SHEQ", "safety health environment quality",
    "SHE", "safety health environment",
    "accident prevention", "accident avoidance",
    "safety record", "safety performance", "TRIR", "LTIR",
    "near-miss", "near miss incident",
    "incident", "accident", "occupational accident",
    "safety audit", "safety audit report",
    "safety protocol", "safety procedure",
    "PPE", "personal protective equipment",
    "confined space", "hot work", "confined space entry",
    "fire prevention", "emergency response",
    "environmental management", "environmental impact",
    "waste disposal", "waste management", "hazardous waste",
    "emission", "air emission", "water discharge",
    "spill", "spillage", "environmental incident",
    "pollution control", "environmental compliance",
    "environmental clearance", "environmental permit",
    
    # Supply Chain & Logistics
    "supply chain", "supply chain management",
    "material", "material supply",
    "logistics", "logistics planning", "logistics management",
    "material handling", "equipment handling",
    "storage", "warehouse", "inventory management",
    "transportation", "freight", "shipping",
    "port operation", "port handling",
    "customs", "import/export", "documentation",
    "vendor", "vendor management", "vendor selection",
    "equipment supplier", "equipment OEM",
    "long-lead items", "lead time",
    "procurement", "procurement planning",
    
    # Workforce & Skills
    "labor", "labour", "manpower", "workforce",
    "skilled workforce", "skilled labor", "skilled worker",
    "unskilled labor", "semi-skilled", "skilled technician",
    "engineer", "project engineer", "field engineer",
    "training", "training program", "operator training",
    "skill development", "capacity building",
    "technical training", "safety training",
    "certification", "competency", "competency assessment",
    "recruitment", "staffing", "labor deployment",
    "welfare", "labor welfare", "working condition",
    
    # Financial & Investment
    "capex", "capital expenditure", "investment",
    "opex", "operational expenditure", "operating cost",
    "project cost", "total project cost", "TPC",
    "financing", "project financing", "infrastructure financing",
    "debt", "equity", "funding",
    "return on investment", "ROI",
    "IRR", "internal rate of return",
    "NPV", "net present value",
    "payback period", "break-even",
    "PPA", "Power Purchase Agreement",
    "take or pay", "offtake agreement",
    "price", "unit price", "price escalation",
    
    # Market & Commercial
    "order intake", "order book", "order backlog",
    "contract value", "project value",
    "YTD", "year-to-date", "FY", "financial year",
    "revenue", "revenue growth", "revenue stream",
    "business segment", "business division",
    "market share", "competitive position",
    "customer", "client", "major client", "marquee client",
    "repeat order", "follow-on order", "repeat business",
    "strategic alliance", "partnership", "consortium",
    "joint venture", "JV", "collaboration",
    
    # Major Oil & Gas Companies - Middle East
    "Saudi Aramco", "Saudi", "KSA", "Kingdom of Saudi Arabia",
    "Saudi oil", "Saudi gas", "Saudi projects",
    "Saudi Aramco projects", "Saudi Aramco orders",
    "SATORP", "Saudi Sinopec", "Saudi Aramco Total",
    "Ras Tanura", "Yanbu", "Safaniyah",
    
    # Major Oil & Gas Companies - UAE & Abu Dhabi
    "UAE", "United Arab Emirates", "Abu Dhabi",
    "ADNOC", "Abu Dhabi National Oil Company",
    "ADNOC Group", "ADNOC Distribution",
    "Al Hosn Gas", "Abu Dhabi Gas Industries",
    "Borouge", "Abu Dhabi Polymers",
    "GASCO", "General Holding Corporation",
    "ENOC", "Emirates National Oil Company",
    
    # Major Oil & Gas Companies - Qatar
    "Qatar", "Doha",
    "QatarEnergy", "Qatar Energy",
    "RasGas", "Ras Gas", "Ras Laffan",
    "Qatargas", "Qatar Gas",
    "LNG", "Qatar LNG", "Qatar liquefied gas",
    "Pearl GTL", "Gas to Liquids",
    
    # Major Oil & Gas Companies - Oman & Kuwait
    "Oman", "PDO", "Petroleum Development Oman",
    "Kuwait", "KOC", "Kuwait Oil Company",
    "KPC", "Kuwait Petroleum Corporation",
    "KNPC", "Kuwait National Petroleum Company",
    
    # Major Oil & Gas Companies - Egypt & Iraq
    "Egypt", "Egyptian oil", "Suez",
    "EGPC", "Egyptian General Petroleum Corporation",
    "Egyptian oil fields", "Western Desert",
    "Iraq", "Basra", "Kirkuk",
    "Iraqi oil", "oil fields",
    "INOC", "Iraqi National Oil Company",
    "Rumaila field", "Majnoon field",
    
    # Global EPC & Service Companies
    "Saipem", "Saipem FPSO", "Saipem pipeline",
    "TechnipFMC", "Technip Energies",
    "Worley", "Worley Services",
    "Petrofac", "Petrofac FPSO",
    "Halliburton", "Schlumberger", "Baker Hughes",
    "Subsea 7", "subsea contractor",
    "Deep Blue", "subsea services",
    "Helix Well Ops", "well services",
    
    # Deepwater Fields & Projects
    "deepwater field", "deepwater project",
    "subsea field", "field development",
    "field infrastructure", "topsides",
    "production system", "storage system", "offloading system",
    "processing equipment", "separation equipment",
    "compressor", "pump", "turbine",
    "control system", "instrumentation",
    "process safety system", "safety system",
    
    # Emerging Challenges & Terminology
    "energy transition", "renewable energy", "net-zero",
    "carbon capture", "CCS", "carbon storage",
    "hydrogen energy", "hydrogen production",
    "biofuel", "biomass", "circular economy",
    "digital transformation", "digitalization",
    "AI", "artificial intelligence", "machine learning",
    "IoT", "Internet of Things", "sensors",
    "predictive maintenance", "condition monitoring",
    "remote operation", "autonomous operation",
    
    # Regional Projects & Terminology
    "pan-India", "all-India", "national level",
    "grassroot facility", "greenfield", "brownfield", "expansion",
    "state-owned enterprise", "SOE", "government PSU",
    "private sector", "private enterprise",
    "domestic market", "international market",
    "export-oriented", "domestic consumption",
    "import substitution", "Atmanirbhar Bharat",
    "Make in India", "local content", "local supplier",
    
    # Legal & Regulatory
    "regulatory body", "government agency",
    "government approval", "ministerial approval",
    "permit", "license", "environmental permit",
    "environmental clearance", "EC", "forest clearance",
    "CRZ", "Coastal Regulation Zone", "CRZ approval",
    "local regulations", "state regulations",
    "compliance requirement", "compliance documentation",
    "public consultation", "stakeholder engagement",
    "environmental impact assessment", "EIA",
    "social impact assessment", "SIA",
    "land acquisition", "land requirement",
    "right of way", "RoW", "easement",
    
    # Operational Metrics & KPIs
    "production rate", "throughput",
    "capacity utilization", "utilization rate",
    "uptime", "downtime", "availability",
    "reliability", "operability", "maintainability",
    "efficiency", "thermal efficiency", "conversion efficiency",
    "cost per unit", "unit cost", "$/bbl", "$/ton",
    "operating cost", "maintenance cost", "lifecycle cost",
    "CAGR", "compound annual growth rate",
    "margin", "profit margin", "operating margin",
    "ROE", "return on equity", "ROCE", "return on capital"
    ],

    "Renewables": [
# Solar Technology & Components
"solar", "photovoltaic", "PV", "solar panel", "module", "solar farm", "solar park", "solar plant", "solar installation", "solar project", "SPV", "solar array", "solar power plant", "utility-scale solar", "ground-mounted solar", "rooftop solar", "roof-top", "floating solar", "FSPV", "floatovoltaics", "canal-top solar", "agrivoltaics", "agrisolar", "agrophotovoltaics", "dual-use solar", "solar sharing", "bifacial solar"

#Solar Capacity & Generation
"capacity MW", "megawatt", "GW", "gigawatt", "kilowatt", "kW", "kWp", "MWp", "generation capacity", "installed capacity", "solar irradiance", "sunlight", "insolation", "capacity factor", "plant load factor", "PLF", "capacity utilization factor", "CUF", "full load hours", "FLH", "energy yield", "performance ratio", "PR", "specific yield", "kWh/kWp"

#Module Technology
"mono-crystalline", "polycrystalline", "poly-crystalline", "multi-crystalline", "thin film", "PERC", "bifacial", "bifacial module", "half-cut cell", "TOPCon", "HJT", "IBC", "CIGS", "CdTe", "N-type", "P-type", "module efficiency", "degradation", "PID", "LID", "hot spot", "bypass diode", "junction box", "glass-glass module", "double glass", "frameless module"

#Balance of System (BoS)
"inverter","BOS","BOP","Balance of system","Balance of plant", "string inverter", "central inverter", "micro-inverter", "power optimizer", "power converter", "DC-AC converter", "maximum power point tracking", "MPPT", "power conditioning unit", "PCU", "transformer", "step-up transformer", "duty transformer", "IDT", "power transformer", "distribution transformer", "33/11 kV", "11/0.4 kV", "switchgear", "ring main unit", "RMU", "disconnect switch", "isolator", "circuit breaker", "ACB", "VCB", "earthing", "grounding", "lightning arrester", "surge protection", "SPD", "combiner box", "DC combiner", "DCDB", "ACDB"

#Cabling & Connections
"cables", "HT cable", "LT cable", "MV cable", "conductor", "underground cable", "overhead line", "DC cable", "AC cable", "solar cable", "PV cable", "armored cable", "XLPE cable", "busbar", "cable tray", "conduit", "cable laying", "cable trench"

#Mounting & Structures
"mounting structure", "mounting system", "racking system", "module mounting structure", "MMS", "foundation", "piling", "driven pile", "rammed pile", "screw pile", "civil foundation", "tracker", "single-axis tracker", "horizontal single-axis tracker", "HSAT", "dual-axis tracker", "fixed mount", "fixed tilt", "tilt angle", "azimuth angle", "row spacing", "pitch", "galvanized steel", "hot-dip galvanized", "HDG", "module clamp", "rail", "purlin", "torque tube"

#Floating Solar Specific
"pontoon", "HDPE pontoon", "floating platform", "buoyancy", "anchoring", "mooring", "anchor block", "mooring line", "walkway", "floating bridge", "reservoir", "water body", "dam", "hydroelectric dam", "water treatment pond", "industrial pond", "lake", "lagoon", "evaporative cooling", "humidity resistant", "corrosion resistant", "anti-corrosion"

#Wind Core Technology
"wind energy", "wind power", "wind farm", "wind project", "wind park", "onshore wind", "offshore wind", "repowering", "wind turbine", "WT", "wind generator", "WTG", "wind machine", "wind turbine capacity"

#Wind Turbine Components
"rotor", "blade", "rotor blade", "hub", "nacelle", "tower", "tubular tower", "lattice tower", "gearbox", "generator", "direct drive", "DFIG", "PMSG", "power electronics", "converter", "yaw system", "pitch control", "brake system", "main shaft", "bearing", "cooling system"

#Wind Performance & Assessment
"capacity factor", "full load hours", "FLH", "productivity", "efficiency", "wind speed", "mean wind speed", "cut-in speed", "cut-out speed", "rated wind speed", "site wind characteristics", "wind resource", "wind resource assessment", "WRA", "anemometer", "wind vane", "meteorological mast", "met mast", "LIDAR", "SODAR", "wind measurement campaign", "wind assessment", "pre-construction survey", "micro-siting", "wind mapping", "wind shear", "turbulence intensity", "wake effect", "power curve", "thrust curve", "availability", "energy production estimate", "P50", "P90"

#Wind Balance of Plant
"balance of plant", "BoP", "WTG foundation", "gravity foundation", "piled foundation", "pad foundation", "internal cabling", "external cabling", "33 kV switchyard", "pooling station", "substation", "grid connection", "interconnection", "transmission line", "access road", "crane pad", "hardstand", "laydown area"

#Battery Storage Core
"battery storage", "energy storage", "BESS", "battery energy storage system", "BES", "ESS", "standalone storage", "co-located storage", "utility-scale storage", "grid-scale storage", "behind-the-meter storage", "BTM", "front-of-meter storage", "FTM"

#Battery Technologies
"lithium-ion", "Li-ion", "LFP", "NMC", "NCA", "lead-acid", "VRLA", "flow battery", "vanadium redox", "zinc-bromine", "sodium-ion", "solid-state battery", "battery cell", "battery module", "battery pack", "battery rack", "battery container", "containerized BESS", "battery management system", "BMS", "thermal management", "HVAC", "liquid cooling", "air cooling", "fire suppression", "fire detection"

#Storage Capacity & Performance
"MWh", "megawatt-hour", "energy capacity", "power capacity", "C-rate", "discharge time", "discharge duration", "1-hour", "2-hour", "4-hour", "charging", "discharging", "charge-discharge cycle", "round-trip efficiency", "RTE", "losses", "auxiliary consumption", "degradation", "state of health", "SOH", "state of charge", "SOC", "depth of discharge", "DOD", "cycle life", "calendar life", "warranty", "throughput", "energy throughput"

#Storage Power Conversion
"rectifier", "power conversion system", "PCS", "bidirectional inverter", "power conversion", "DC-AC", "AC-DC", "grid synchronization", "grid forming", "grid following"

#Other Storage Technologies
"compressed air", "CAES", "pumped hydro", "PSH", "flywheel", "thermal storage", "molten salt", "green hydrogen storage", "hydrogen electrolyzer"

#Hybrid Configurations
"hybrid system", "hybrid project", "solar + storage", "solar-plus-storage", "wind + storage", "wind-plus-storage", "solar + wind", "solar-wind hybrid", "solar + wind + storage", "renewable energy + storage", "RE + storage", "firm renewable", "FDRE","firm power", "dispatchable renewable", "24/7 clean energy", "round-the-clock", "RTC power", "co-location", "co-located hybrid", "standalone hybrid"

#Grid Connection
"grid connection", "grid interconnection", "interconnection", "evacuation", "power evacuation", "transmission connection", "distribution network", "pooling station", "substation", "switchyard", "bay", "feeder", "transmission line", "66 kV", "110 kV", "132 kV", "220 kV", "400 kV", "HVAC", "HVDC", "STU", "CTU", "ISTS", "intra-state", "inter-state"

#Grid Services & Stability
"peak shaving", "peak load management", "grid support", "frequency regulation", "frequency response", "primary frequency response", "PFR", "voltage support", "voltage regulation", "reactive power", "MVAR", "power factor", "ancillary services", "grid stability", "grid reliability", "grid balancing", "load balancing", "demand response", "demand-side management", "DSM", "ramping", "ramp rate", "black start", "islanding", "anti-islanding"

#Grid Monitoring & Control
"SCADA", "EMS", "DMS", "smart grid", "smart inverter", "advanced inverter", "digital monitoring", "remote monitoring", "telemetry", "data analytics", "predictive maintenance", "condition monitoring", "asset management"

#Variable Renewables
"variable renewable", "VRE", "intermittency", "intermittent energy", "variability", "forecasting", "wind forecasting", "solar forecasting", "generation forecasting", "day-ahead forecast", "intra-day forecast", "scheduling", "curtailment", "must-run"

#Hydrogen Production
"green hydrogen", "GH2", "hydrogen", "H2", "renewable hydrogen", "clean hydrogen", "electrolysis", "electrolyzer", "water electrolysis", "alkaline electrolyzer", "AE", "PEM electrolyzer", "PEMEC", "solid oxide electrolyzer", "SOEC", "anion exchange membrane", "AEM", "stack", "electrolyzer stack", "electrolyzer capacity","electrolyser", "water electrolysis", "alkaline electrolyser", "AE", "PEM electrolyser", "PEMEC", "solid oxide electrolyser","electrolyser stack", "electrolyser capacity"

#Hydrogen Infrastructure
"hydrogen production", "hydrogen storage", "hydrogen compression", "compressor", "hydrogen transport", "hydrogen pipeline", "hydrogen refueling", "hydrogen dispensing", "fuel cell", "PEM fuel cell", "SOFC", "hydrogen blending", "power-to-gas", "P2G", "power-to-X", "P2X", "hydrogen derivatives", "green ammonia", "green methanol"

#Hydrogen Standards & Emission
"well-to-gate emission", "carbon intensity", "2 kg CO2 equivalent", "emission threshold", "hydrogen certification", "guarantee of origin", "GO"

#Bidding & Procurement
"capacity auction", "tariff auction", "competitive bidding", "reverse auction", "auction","e-reverse auction", "tender", "RFQ", "RFP", "EOI", "RFS", "bid document", "bidding document", "technical bid", "financial bid", "price bid", "techno-commercial bid", "two-envelope system", "single-stage", "two-stage", "pre-qualification", "pre-bid meeting", "bid submission", "bid evaluation", "L1", "award", "letter of award", "LOA", "contract", "agreement", "signing"

#Contracts & Agreements
"power purchase agreement", "PPA", "offtake", "offtake agreement", "energy sale agreement", "ESA", "long-term PPA", "LTPPA", "EPC contract", "EPC agreement", "turnkey contract", "lump-sum turnkey", "LSTK", "BOT", "BOOT", "O&M contract", "operation and maintenance agreement", "performance-based O&M", "comprehensive O&M", "AMC", "RLDC", "SLDC", "PSA", "land lease agreement", "transmission service agreement", "TSA", "interconnection agreement"

#Project Finance & Economics
"independent power producer", "IPP", "renewable energy developer", "developer", "project developer", "SPV", "project SPV", "project finance", "non-recourse financing", "limited recourse", "CAPEX", "OPEX", "project cost", "total project cost", "levelized cost", "LCOE", "LCOS", "tariff", "discovered tariff", "bid tariff", "contract tariff", "Rs/kWh", "IRR", "equity IRR", "project IRR", "ROE", "DSCR", "payback period", "NPV", "bankability", "bankable", "financial closure", "debt financing", "equity", "senior debt", "subordinated debt", "mezzanine", "term loan", "working capital"

#Development & Approvals
"project development", "site identification", "land acquisition", "land procurement", "lease", "land lease", "land area", "hectare", "acre", "environmental clearance", "EC", "environmental impact assessment", "EIA", "forest clearance", "FC", "wildlife clearance", "CRZ clearance", "statutory clearances", "permits", "approvals", "consents", "NOC", "right of way", "ROW", "way leave", "geotechnical investigation", "soil investigation", "geo-tech study", "topographical survey", "topo survey", "drone survey", "LiDAR survey", "bathymetric survey", "hydrological study", "hydrology", "wind study", "solar resource study", "irradiation study", "GHI", "DNI", "DHI", "GTI"

#Engineering & Design
"DPR", "feasibility study", "techno-economic feasibility", "pre-feasibility", "project report", "engineering", "detailed engineering", "basic engineering", "FEED", "design", "system design", "electrical design", "civil design", "structural design", "SLD", "layout", "plant layout", "array layout", "WTG layout", "general arrangement", "GA drawing", "technical specification", "equipment specification", "bill of material", "BOM", "bill of quantity", "BOQ", "material take-off", "MTO"

#Procurement & Supply Chain
"procurement", "supply", "supply chain", "supply chain management", "sourcing", "vendor", "supplier", "OEM", "manufacturer", "purchase order", "PO", "delivery", "lead time", "logistics", "transportation", "freight", "import", "export", "customs clearance", "port", "port handling", "inland transportation", "ODC", "abnormal load", "special cargo"

#Construction & Installation
"EPC", "EPCM", "turnkey", "construction", "civil works", "civil construction", "earthwork", "grading", "leveling", "compaction", "excavation", "concrete works", "RCC", "PCC", "steel structure", "fabrication", "erection", "installation", "module installation", "WTG erection", "tower erection", "blade installation", "mechanical works", "electrical works", "cabling works", "termination", "jointing", "cable pulling", "cable laying", "commissioning", "pre-commissioning", "testing", "testing and commissioning", "T&C", "energization", "synchronization", "trial run", "performance test", "performance guarantee test", "PG test", "handover", "provisional handover", "final handover", "takeover", "COD", "SCOD", "commissioning certificate"

#Project Management
"project management", "project execution", "PMC", "owner's engineer", "IE", "lender's engineer", "construction supervision", "site supervision", "progress monitoring", "milestone", "critical path", "CPM", "Gantt chart", "project schedule", "timeline", "delay", "extension of time", "EOT", "liquidated damages", "LD", "penalty", "bonus", "incentive", "performance-based incentive"

#O&M Activities
"operation", "maintenance", "O&M", "operation and maintenance", "preventive maintenance", "PM", "corrective maintenance", "breakdown maintenance", "predictive maintenance", "condition-based maintenance", "CBM", "scheduled maintenance", "unscheduled maintenance", "major maintenance", "minor maintenance", "periodic maintenance", "annual maintenance", "service", "servicing", "inspection", "routine inspection", "cleaning", "module cleaning", "robotic cleaning", "water-less cleaning", "panel washing", "vegetation management", "solar grazing", "sheep grazing", "weed control"

#Performance & Monitoring
"plant performance", "performance monitoring", "generation", "energy generation", "power generation", "output", "availability", "plant availability", "uptime", "downtime", "forced outage", "planned outage", "troubleshooting", "fault finding", "root cause analysis", "RCA", "performance optimization", "yield optimization", "soiling loss", "shading loss", "module mismatch", "inverter clipping", "curtailment loss", "grid unavailability", "transmission loss"

#Warranty & Insurance
"warranty", "module warranty", "inverter warranty", "WTG warranty", "comprehensive warranty", "defects liability period", "DLP", "performance guarantee", "PG", "performance bank guarantee", "PBG", "insurance", "project insurance", "all-risk insurance", "business interruption insurance", "BI", "loss of profit", "LOP", "risk management", "contingency", "force majeure"

#National Policies & Targets
"renewable purchase obligation", "RPO", "solar RPO", "non-solar RPO", "hydro purchase obligation", "HPO", "renewable energy certificate", "REC", "green certificate", "subsidy", "incentive", "viability gap funding", "VGF", "capital subsidy", "generation-based incentive", "GBI", "tax credit", "tax benefit", "accelerated depreciation", "AD", "renewable energy target", "500GW target", "500 GW by 2030", "450 GW", "national mission", "National Solar Mission", "NSM", "Jawaharlal Nehru National Solar Mission", "JNNSM", "state policy", "solar mission", "wind mission", "storage mission", "National Green Hydrogen Mission", "NGHM", "energy storage obligation", "ESO", "hybrid tender", "ISTS waiver", "transmission charges waiver", "banking", "wheeling", "open access", "captive consumption", "group captive", "third-party sale"

#Regulatory Bodies
"MNRE", "Ministry of Power", "MOP", "Ministry of Environment", "MOEF&CC", "Central Electricity Authority", "CEA", "CERC", "SERC", "SECI", "NTPC Renewable Energy", "NTPC-REL", "IREDA", "PFC", "REC Limited", "PGCIL", "state nodal agency", "SNA", "DISCOM", "electricity board", "state utility", "central PSU", "CPSU"

#Standards & Specifications
"IEC standards", "IEC 61215", "IEC 61730", "IEC 61400", "IEC 62933", "IEEE standards", "IEEE 1547", "Indian standards", "IS standards", "BIS", "MNRE specifications", "MNRE technical standards", "CEA regulations", "CEA technical standards", "grid code", "Indian Electricity Grid Code", "IEGC", "state grid code", "ALMM", "type approval", "type testing", "certification", "third-party inspection", "TPI", "quality assurance", "QA", "quality control", "QC", "local regulations"

#Safety, Health & Environment
"safety", "health", "environment", "EHS", "SHEQ", "HSE", "safety management", "safety protocol", "PPE", "risk assessment", "hazard identification", "HIRA", "incident", "accident", "near miss", "LTI", "LTIFR", "zero harm", "safety training", "toolbox talk", "safety induction", "permit to work", "PTW", "hot work permit", "work at height", "electrical safety", "fire safety", "emergency response", "first aid", "environmental management", "environmental compliance", "dust suppression", "noise control", "waste management", "soil erosion", "water conservation", "biodiversity", "ecological", "carbon footprint", "carbon neutral", "net zero", "ESG", "sustainability", "sustainable development"

#Labor & Workforce
"skilled labor", "skilled workforce", "technician", "engineer", "supervisor", "project manager", "site engineer", "electrical engineer", "civil engineer", "O&M technician", "commissioning engineer", "testing engineer", "training", "skill development", "certification", "manpower", "labor", "contractor", "subcontractor", "workforce planning", "resource mobilization"

#Power Market
"power trading", "energy trading", "merchant power", "merchant plant", "bilateral contract", "bilateral PPA", "short-term market", "short-term PPA", "day-ahead market", "DAM", "real-time market", "RTM", "term-ahead market", "TAM", "green market", "green DAM", "green term-ahead market", "G-TAM", "power exchange", "IEX", "PXIL", "HPX", "market clearing price", "MCP", "ancillary services market", "capacity market"

#Public Sector Competitors
"NTPC", "NTPC Renewable Energy", "NTPC-REL", "NTPC Green Energy", "SECI", "SJVN", "SJVN Green", "NHPC", "THDC India", "state DISCOM", "state utility"

#Private Sector Developers
"Adani Green Energy", "Adani Renewable", "AGEL", "Adani Solar", "ReNew Power", "ReNew Energy", "Tata Power Renewable", "Tata Power Solar", "Greenko", "Azure Power", "Hero Future Energies", "Avaada Energy", "Ayana Renewable", "Waaree", "Vikram Solar", "Sterling and Wilson", "JSW Energy", "JSW Neo Energy", "Torrent Power", "CLP India", "Sembcorp", "Acme Solar", "Sprng Energy", "O2 Power", "Fourth Partner Energy"

#EPC Contractors
"KEC International", "Larsen & Toubro", "L&T", "Sterling and Wilson Renewable Energy", "SWREL", "Tata Projects", "Kalpataru Power Transmission", "KPTL", "Reliance Infrastructure", "BGR Energy", "Jyoti Structures", "Mahindra Susten"

#International Players
"First Solar", "Longi Solar", "Jinko Solar", "Canadian Solar", "Trina Solar", "JA Solar", "Risen Energy", "Huawei", "Sungrow", "SMA Solar", "Siemens Gamesa", "Vestas", "GE Renewable Energy", "Suzlon", "Nordex", "Goldwind", "Envision"

#Emerging Technologies & Trends 2025
"AI in renewable energy", "artificial intelligence", "machine learning", "ML", "drone inspection", "UAV inspection", "thermography", "infrared inspection", "IR imaging", "blockchain", "distributed ledger", "IoT", "smart sensor", "edge computing", "5G", "digital twin", "virtual power plant", "VPP", "peer-to-peer trading", "P2P", "energy-as-a-service", "EaaS", "carbon credit", "carbon offset", "carbon trading", "voluntary carbon market", "compliance carbon market", "perovskite", "perovskite solar cell", "tandem solar cell", "building-integrated photovoltaic", "BIPV", "vehicle-to-grid", "V2G", "smart charging", "recycling", "circular economy", "second life battery", "repurposing"    ],

    "Global": [    # Core EPC & Project Terms
    "EPC", "Engineering Procurement Construction",
    "EPCI", "Engineering Procurement Construction Installation",
    "EPCM", "Engineering Procurement Construction Management",
    "EPIC", "Engineering Procurement Installation Construction",
    "engineering services", "procurement services", "construction services",
    "turnkey", "turnkey project", "turnkey solution",
    "complete package", "end-to-end solution", "end-to-end delivery",
    "integrated project delivery", "IPD",
    
    # Contract & Order Terms
    "contract", "contract award", "contract win",
    "contract value", "project value", "order value",
    "contract signing", "contract execution",
    "bid win", "bid success", "order win",
    "order intake", "order book", "order backlog",
    "L1", "L1 position", "L1 order", "first loss",
    "YTD", "year-to-date", "FY", "financial year",
    "order pipeline", "tender pipeline", "pipeline value",
    "order announcement", "press release", "official statement",
    "deal size", "deal value", "contract size",
    
    # Tendering & Bidding
    "tender", "competitive tender", "open tender",
    "RFQ", "Request for Quotation",
    "RFP", "Request for Proposal",
    "EOI", "Expression of Interest",
    "pre-bid", "pre-bid meeting",
    "bid", "bidding", "competitive bidding",
    "bid submission", "bid deadline",
    "technical bid", "commercial bid", "financial bid",
    "bid evaluation", "bid committee",
    "bid winner", "bid loser",
    "award", "contract award",
    "LOA", "Letter of Award",
    "LOI", "Letter of Intent",
    "MOU", "Memorandum of Understanding",
    "placement", "shortlist", "shortlisted",
    
    # Project Execution & Delivery
    "execution", "project execution", "implementation",
    "project delivery", "project management", "PM",
    "delivery phase", "execution phase", "commissioning phase",
    "project planning", "execution planning",
    "schedule", "project schedule", "execution schedule",
    "timeline", "project timeline",
    "milestone", "major milestone", "key milestone",
    "progress", "physical progress", "financial progress",
    "completion", "project completion",
    "completion date", "scheduled completion",
    "COD", "Commercial Operation Date",
    "handover", "project handover", "operational handover",
    "final handover", "provisional handover",
    
    # Quality, Standards & Compliance
    "quality assurance", "QA", "quality control", "QC",
    "inspection", "third-party inspection", "TPI",
    "testing", "site testing", "performance test",
    "commissioning", "testing and commissioning", "T&C",
    "pre-commissioning", "pre-commissioning inspection",
    "certification", "certification process",
    "compliance", "regulatory compliance",
    "standards", "international standards", "industry standards",
    "ISO", "ISO certification", "ISO compliance",
    "API", "ASME", "DNV", "ABS", "classification society",
    "code of practice", "design code",
    "specification", "technical specification",
    "performance standard", "performance guarantee",
    "warranty", "warranty period",
    
    # Health, Safety & Environment
    "HSE", "health safety environment",
    "SHEQ", "safety health environment quality",
    "EHS", "environment health safety",
    "safety", "safety management", "safety protocol",
    "health", "occupational health",
    "environment", "environmental management",
    "TRIR", "total recordable incident rate",
    "LTIR", "lost time incident rate",
    "accident", "accident rate", "accident prevention",
    "near-miss", "near miss incident",
    "incident", "incident reporting", "incident investigation",
    "safety audit", "safety audit report",
    "environmental clearance", "EC",
    "environmental impact assessment", "EIA",
    "social impact assessment", "SIA",
    
    # Project Management & Finance
    "project management", "PMC", "project manager",
    "cost", "project cost", "cost estimate",
    "budget", "project budget", "budgeting",
    "capex", "capital expenditure",
    "opex", "operational expenditure",
    "investment", "infrastructure investment",
    "financing", "project financing", "infrastructure financing",
    "cost overrun", "budget overrun",
    "schedule overrun", "time overrun", "delay",
    "contingency", "contingency reserve",
    "risk management", "risk assessment", "risk mitigation",
    "insurance", "project insurance",
    "performance guarantee", "performance bond",
    "liquidated damages", "LD", "delay charges",
    
    # Operations & Maintenance
    "operations", "project operations",
    "O&M", "operation and maintenance",
    "maintenance", "scheduled maintenance",
    "support", "technical support", "project support",
    "warranty", "post-warranty",
    "performance", "operational performance",
    "efficiency", "operational efficiency",
    "reliability", "system reliability",
    "availability", "system availability", "uptime",
    "downtime", "maintenance downtime",
    
    # Business & Market Metrics
    "revenue", "revenue growth", "revenue stream",
    "profitability", "profitable growth",
    "margin", "EBITDA margin", "operating margin", "profit margin",
    "EBITDA", "earnings before interest tax depreciation",
    "PAT", "profit after tax", "net profit",
    "PBT", "profit before tax",
    "EPS", "earnings per share",
    "ROE", "return on equity",
    "ROI", "return on investment",
    "IRR", "internal rate of return",
    "NPV", "net present value",
    "order book value", "backlog value",
    "visible cash flow", "revenue visibility",
    "growth rate", "CAGR", "compound annual growth rate",
    
    # Company & Financial Indicators
    "financial performance", "quarterly results", "annual results",
    "stock price", "share price", "stock valuation",
    "market capitalization", "market cap", "market value",
    "PE ratio", "price-to-earnings", "PB ratio", "price-to-book",
    "dividend", "dividend yield", "dividend payment",
    "earnings report", "earnings call",
    "investor relations", "IR", "investor update",
    "analyst rating", "analyst report",
    "credit rating", "bond rating",
    "debt", "leverage", "debt-to-equity",
    "cash flow", "free cash flow", "FCF",
    "balance sheet", "income statement", "cash flow statement",
    
    # Corporate Governance & Strategy
    "management", "board of directors", "leadership",
    "CEO", "Chief Executive Officer", "MD", "Managing Director",
    "CFO", "Chief Financial Officer",
    "strategic direction", "strategic focus",
    "business strategy", "growth strategy",
    "competitive advantage", "market position",
    "market share", "market leadership",
    "competitive landscape", "competitive analysis",
    "market trends", "industry trends",
    "opportunity", "market opportunity",
    "threat", "competitive threat", "market threat",
    "SWOT", "SWOT analysis",
    
    # Business Segments & Diversification
    "business segment", "business division", "business unit",
    "segment performance", "segment revenue",
    "business portfolio", "portfolio diversification",
    "geographic diversification", "geographic presence",
    "product diversification", "service diversification",
    "new business", "new market entry",
    "emerging market", "developing market",
    "tier-I city", "tier-II city", "metro", "non-metro",
    "pan-India presence", "all-India", "national presence",
    
    # Competitor Analysis
    "competitor", "competitor analysis", "competitive intelligence",
    "major player", "key player", "market leader",
    "new entrant", "emerging player",
    "competitive bidding", "competitive position",
    "market share", "share gains", "share loss",
    "market consolidation", "M&A", "merger and acquisition",
    "acquisition", "takeover", "buyout",
    "strategic alliance", "partnership", "collaboration",
    "joint venture", "JV", "consortium",
    "subcontract", "subcost", "sub-supply",
    
    # Technology & Innovation
    "technology", "technological capability",
    "innovation", "innovative solution",
    "R&D", "research and development",
    "digital transformation", "digitalization", "digital"
    "automation", "process automation",
    "IoT", "Internet of Things",
    "AI", "artificial intelligence", "machine learning",
    "BIM", "Building Information Modeling",
    "digital twin", "digital model",
    "software", "platform", "digital platform",
    "ERP", "enterprise resource planning",
    "technology investment", "tech capex",
    "technology adoption", "emerging technology",
    "cybersecurity", "data security",
    
    # Sustainability & ESG
    "sustainability", "sustainable", "sustainable growth",
    "ESG", "environmental social governance",
    "green", "green project", "green business",
    "renewable", "renewable energy", "renewable project",
    "net-zero", "zero-carbon", "carbon neutral",
    "low-carbon", "carbon reduction",
    "emissions", "emissions reduction",
    "carbon footprint", "carbon intensity",
    "climate change", "climate action",
    "environmental responsibility", "social responsibility",
    "community engagement", "stakeholder engagement",
    "CSR", "corporate social responsibility",
    "sustainable development", "SDG", "sustainable development goals",
    "decarbonization", "energy transition",
    
    # Global & International Operations
    "international", "global", "worldwide",
    "international project", "global project",
    "international presence", "global presence",
    "overseas", "overseas market", "overseas project",
    "export", "export market", "export order",
    "import", "import substitution",
    "cross-border", "transnational",
    "geographic expansion", "international expansion",
    "new market", "emerging market", "developing economy",
    "developed market", "mature market",
    "regional presence", "country presence",
    "footprint", "market footprint",
    "supply chain", "global supply chain",
    "local content", "local supplier", "local partner",
    "technology transfer", "knowledge transfer",
    
    # India-Specific Terms
    "India", "Indian market", "domestic market",
    "Make in India", "Atmanirbhar Bharat",
    "national infrastructure", "infrastructure development",
    "government projects", "government order",
    "state government", "central government",
    "PSU", "public sector undertaking",
    "ministry", "ministry project",
    "national plan", "infrastructure program",
    "Smart Cities Mission", "Bharatmala",
    "Delhi-Mumbai Expressway", "national highway",
    "state capital", "tier-I city", "tier-II city",
    "urban development", "urban infrastructure",
    "industrial corridor", "economic zone",
    
    # Media & Communication
    "press release", "official announcement",
    "news", "company news", "corporate news",
    "media report", "news report",
    "earnings call", "conference call",
    "investor presentation", "annual report",
    "quarterly report", "quarterly results",
    "stock exchange filing", "regulatory filing",
    "regulatory announcement", "stock exchange announcement",
    "management commentary", "management view",
    "market update", "industry update",
    
    # Strategic Initiatives & Priorities
    "strategic initiative", "key initiative",
    "growth initiative", "transformation initiative",
    "digital initiative", "sustainability initiative",
    "efficiency initiative", "cost reduction",
    "revenue growth", "margin expansion",
    "cash generation", "working capital optimization",
    "organizational restructuring", "restructuring",
    "capacity expansion", "capacity addition",
    "capability building", "capability enhancement",
    "team expansion", "recruitment", "talent acquisition",
    "training", "skill development", "capacity building",
    
    # Regional Markets (KEC Focus Areas)
    "India", "Middle East", "GCC", "Saudi Arabia",
    "UAE", "Africa", "Southeast Asia", "ASEAN",
    "South Asia", "Central Asia", "Europe",
    "Latin America", "Americas", "North America",
    "Asia-Pacific", "Asia", "Pacific", "PNG",
    "Australia", "New Zealand",
    
    # Industry Segments Served by KEC
    "power", "power transmission", "power distribution",
    "railways", "rail infrastructure", "metro",
    "civil infrastructure", "highways", "roads",
    "oil and gas", "pipelines", "refinery",
    "renewable energy", "solar", "wind",
    "transmission", "distribution", "T&D",
    "cables", "cable business",
    "smart infrastructure", "smart grid",
    "data center", "telecommunications",
    
    # Partnership & Collaboration
    "joint venture", "JV", "JV partner",
    "consortium", "consortium partner",
    "partnership", "strategic partner",
    "alliance", "strategic alliance",
    "collaboration", "collaborative project",
    "merger", "acquisition", "M&A",
    "integration", "post-merger integration",
    "takeover", "buyout",
    
    # Regulatory & Governance
    "regulatory", "regulatory approval",
    "government approval", "ministerial approval",
    "permit", "license", "licensing",
    "compliance", "regulatory compliance",
    "policy", "government policy",
    "tariff", "pricing regulation",
    "environmental permit", "environmental clearance",
    "social clearance", "land acquisition",
    "right of way", "RoW",
    "dispute", "contractual dispute", "arbitration",
    
    # Industry Outlook & Trends
    "industry trend", "market trend", "emerging trend",
    "infrastructure spending", "capital expenditure",
    "government capex", "capex cycle",
    "order cycle", "project cycle",
    "economic growth", "GDP growth",
    "urbanization", "urban growth",
    "industrialization", "industrial growth",
    "digitalization trend", "automation trend",
    "sustainability trend", "green trend",
    "consolidation", "market consolidation",
    "commoditization", "pricing pressure",
    
    # Key Performance Indicators (KPIs)
    "KPI", "key performance indicator",
    "order book growth", "order intake growth",
    "revenue growth rate", "growth rate",
    "margin improvement", "margin expansion",
    "market share gain", "share performance",
    "operational efficiency", "project efficiency",
    "execution track record", "delivery track record",
    "on-time delivery", "on-budget delivery",
    "project profitability", "project margin",
    "customer satisfaction", "client satisfaction",
    "safety record", "safety performance",
    
    # Challenges & Headwinds
    "challenge", "market challenge",
    "headwind", "economic headwind",
    "currency", "currency fluctuation", "forex",
    "commodity prices", "price volatility",
    "supply chain", "supply chain disruption",
    "inflation", "cost inflation",
    "labor cost", "wage inflation",
    "competition", "intense competition",
    "price pressure", "margin pressure",
    "project delay", "schedule delay",
    "client default", "payment delay",
    "regulatory change", "policy change",
    
    # Future Outlook & Guidance
    "guidance", "management guidance",
    "outlook", "company outlook", "industry outlook",
    "target", "growth target", "margin target",
    "forecast", "revenue forecast", "earnings forecast",
    "FY26", "FY27", "next fiscal", "full year",
    "medium-term", "long-term",
    "FY26 guidance", "FY27 guidance",
    "midpoint", "lower end", "upper end",
    "capital allocation", "dividend policy",
    
    # Deal & Transaction Related
    "order announcement", "contract announcement",
    "major order", "mega order", "large contract",
    "repeat order", "follow-on order", "extension",
    "first order", "maiden order",
    "marquee client", "blue-chip client",
    "high-profile project", "landmark project",
    "signature project", "flagship project",
    
    # Quarterly & Annual Performance Cycles
    "quarterly", "Q1", "Q2", "Q3", "Q4",
    "quarter ended", "half-yearly", "half year", "H1", "H2",
    "annual", "full year", "FY", "fiscal year",
    "YoY", "year-on-year", "sequential", "QoQ",
    "nine months", "nine-month", "YTD",
    "results announcement", "results release",
    "results date", "results call",
    "guidance update", "revised guidance",
    
    # Investor Communication
    "annual report", "annual review",
    "quarterly report", "quarterly update",
    "investor presentation", "investor briefing",
    "investor day", "analyst day",
    "conference call", "earnings call",
    "webinar", "online event",
    "investor relations", "IR communication",
    "corporate governance", "board oversight",
    "transparency", "disclosure", "mandatory disclosure",
    
    # Stock & Valuation Related
    "share", "share price", "stock price",
    "market capitalization", "market cap",
    "share split", "rights issue", "bonus",
    "dividend", "dividend announcement",
    "stock buyback", "share buyback",
    "listing", "delisting", "stock exchange",
    "NSE", "BSE", "ticker", "symbol",
    "ISIN", "stock code",
    "trading volume", "trading liquidity",
    "bid-ask spread", "stock volatility" ]
}

# -------------------------
# 2️⃣ Competitor List
# -------------------------
COMPETITOR_MASTER = {
    # --- India T&D ---
    "Larsen & Toubro Limited": {
        "sbu": "India T&D",
        "aliases": [
            "L&&T", "L&&T's", "L&T", "L&T's", "L&TL", "L&TL's", "L-&-T", "L-&-T's", "L.&.T", "L.&.T's", "Larsen & Toubro", "Larsen & Toubro Lim", "Larsen & Toubro Lim's", "Larsen & Toubro Limited", "Larsen & Toubro Limited's", "Larsen & Toubro Ltd", "Larsen & Toubro Ltd.", "Larsen & Toubro Ltd.'s", "Larsen & Toubro Private Lim", "Larsen & Toubro Private Lim's", "Larsen & Toubro Private Limited", "Larsen & Toubro Private Limited's", "Larsen & Toubro Pvt Ltd", "Larsen & Toubro Pvt Ltd's", "Larsen & Toubro Pvt. Ltd.", "Larsen &T", "Larsen &T's", "Larsen Toubro Lim", "Larsen Toubro Lim's", "Larsen Toubro Limited", "Larsen Toubro Limited's", "Larsen and Toubro Lim", "Larsen and Toubro Limited", "Larsen&T", "Larsen&T's", "Larsen&Toubro", "Larsen&Toubro's", "Larsen&ToubroLim", "Larsen&ToubroLim's", "Larsen&ToubroLimited", "Larsen&ToubroLimited's", "larsen&toubro", "larsen&toubro's","LT","larsen toubro","Larsen Toubro", "larsentoubro","LarsenToubro", "larsen&toubrolimited" ]
    },

    "Kalpataru Projects International Limited": {
        "sbu": "India T&D",
        "aliases": ["KP", "KPIL", "Kalpataru Projects International", "Kalpataru Projects International Limited", "Kalpataru Projects International Ltd", "Kalpataru Projects International Ltd.", "KalpataruProjectsInternationalLimited", "kalpataruprojectsinternationallimited", "KPTL", "Kalpataru", "Kalpataru Projects", "Kalpataruprojects"
        ]
    },

    "Tata Projects Limited": {
        "sbu": "Transmission & EPC",
        "aliases": ["T&P", "T-P", "T-P's", "T.P", "T.P's", "TP", "TPL", "Tata P", "Tata Proj", "Tata Proj Lim", "Tata Proj Lim's", "Tata Proj Limited", "Tata Proj Limited's", "Tata Proj Ltd", "Tata Proj Ltd's", "Tata Proj Ltd.", "Tata Proj Ltd.'s", "Tata Proj Private Lim", "Tata Proj Private Limited", "Tata Proj Pvt Ltd", "Tata Proj Pvt. Ltd.", "Tata Proj Pvt. Ltd.'s", "Tata Proj's", "Tata Proj.", "Tata Proj. Lim", "Tata Proj. Lim's", "Tata Proj. Limited", "Tata Proj. Limited's", "Tata Proj. Ltd", "Tata Proj. Ltd's", "Tata Proj. Ltd.", "Tata Proj. Ltd.'s", "Tata Proj. Private Lim", "Tata Proj. Private Limited", "Tata Proj. Pvt Ltd", "Tata Proj. Pvt. Ltd.", "Tata Proj. Pvt. Ltd.'s", "Tata Proj.'s", "Tata Projects", "Tata Projects Lim", "Tata Projects Lim's", "Tata Projects Limited", "Tata Projects Limited's", "Tata Projects Ltd", "Tata Projects Ltd's", "Tata Projects Ltd.", "Tata Projects Ltd.'s", "Tata Projects Private Lim", "Tata Projects Private Limited", "Tata Projects Pvt Ltd", "Tata Projects Pvt. Ltd.", "Tata Projects Pvt. Ltd.'s", "Tata Projects's", "TataP", "TataP's", "TataProj", "TataProj.", "TataProj.Lim", "TataProj.Limited", "TataProjLim", "TataProjLimited", "TataProjects", "TataProjectsLim", "TataProjectsLimited", "tataprojects", "tataprojects's", "tataprojectslimited", "tataprojectslimited's"        ]
    },

    "Sterlite Power Transmission Limited": {
        "sbu": "India T&D",
        "aliases": ["S&P&T", "S&P&T's", "S-P-T", "S-P-T's", "S.P.T", "SPT", "SPT's", "SPTL", "SPTL's", "Sterlite PT", "Sterlite PT's", "Sterlite Power Transmission", "Sterlite Power Transmission Lim", "Sterlite Power Transmission Lim's", "Sterlite Power Transmission Limited", "Sterlite Power Transmission Limited's", "Sterlite Power Transmission Ltd", "Sterlite Power Transmission Ltd's", "Sterlite Power Transmission Ltd.", "Sterlite Power Transmission Private Lim", "Sterlite Power Transmission Private Limited", "Sterlite Power Transmission Pvt Ltd", "Sterlite Power Transmission Pvt Ltd's", "Sterlite Power Transmission Pvt. Ltd.", "Sterlite Power Transmission Pvt. Ltd.'s", "Sterlite Power Transmission's", "SterlitePT", "SterlitePT's", "SterlitePowerTransmission", "SterlitePowerTransmission's", "SterlitePowerTransmissionLim", "SterlitePowerTransmissionLim's", "SterlitePowerTransmissionLimited", "SterlitePowerTransmissionLimited's", "sterlitepowertransmission", "sterlitepowertransmission's", "sterlitepowertransmissionlimited", "sterlitepowertransmissionlimited's"        ]
    },

    "Bharat Heavy Electricals Limited": {
        "sbu": "India T&D",
        "aliases": [
"B&H&E", "B-H-E", "B.H.E", "B.H.E's", "BHE", "BHE's", "BHEL", "BHEL's", "Bharat HE", "Bharat HE's", "Bharat Heavy Electricals", "Bharat Heavy Electricals Lim", "Bharat Heavy Electricals Lim's", "Bharat Heavy Electricals Limited", "Bharat Heavy Electricals Limited's", "Bharat Heavy Electricals Ltd", "Bharat Heavy Electricals Ltd.", "Bharat Heavy Electricals Ltd.'s", "Bharat Heavy Electricals Private Lim", "Bharat Heavy Electricals Private Lim's", "Bharat Heavy Electricals Private Limited", "Bharat Heavy Electricals Private Limited's", "Bharat Heavy Electricals Pvt Ltd", "Bharat Heavy Electricals Pvt Ltd's", "Bharat Heavy Electricals Pvt. Ltd.", "Bharat Heavy Electricals Pvt. Ltd.'s", "Bharat Heavy Electricals's", "BharatHE", "BharatHE's", "BharatHeavyElectricals", "BharatHeavyElectricals's", "BharatHeavyElectricalsLim", "BharatHeavyElectricalsLim's", "BharatHeavyElectricalsLimited", "BharatHeavyElectricalsLimited's", "bharatheavyelectricals", "bharatheavyelectricals's", "bharatheavyelectricalslimited", "bharatheavyelectricalslimited's"        ]
    },

    "Siemens Energy India Limited": {
        "sbu": "India T&D",
        "aliases": [
            "S&E&I", "S-E-I", "S-E-I's", "S.E.I", "S.E.I's", "SEI", "SEI's", "SEIL", "SEIL's", "Siemens EI", "Siemens EI's", "Siemens Energy India", "Siemens Energy India Lim", "Siemens Energy India Lim's", "Siemens Energy India Limited", "Siemens Energy India Limited's", "Siemens Energy India Ltd", "Siemens Energy India Ltd's", "Siemens Energy India Ltd.", "Siemens Energy India Ltd.'s", "Siemens Energy India Private Lim", "Siemens Energy India Private Lim's", "Siemens Energy India Private Limited", "Siemens Energy India Private Limited's", "Siemens Energy India Pvt Ltd", "Siemens Energy India Pvt Ltd's", "Siemens Energy India Pvt. Ltd.", "Siemens Energy India Pvt. Ltd.'s", "SiemensEI", "SiemensEI's", "SiemensEnergyIndia", "SiemensEnergyIndia's", "SiemensEnergyIndiaLim", "SiemensEnergyIndiaLim's", "SiemensEnergyIndiaLimited", "SiemensEnergyIndiaLimited's", "siemensenergyindia", "siemensenergyindialimited", "siemensenergyindialimited's" ]
    },

    "Hitachi Energy India Limited": {
        "sbu": "India T&D",
        "aliases": [
            "H&E&I", "H&E&I's", "H-E-I", "H-E-I's", "H.E.I", "H.E.I's", "HEI", "HEI's", "HEIL", "Hitachi EI", "Hitachi EI's", "Hitachi Energy India", "Hitachi Energy India Lim", "Hitachi Energy India Lim's", "Hitachi Energy India Limited", "Hitachi Energy India Limited's", "Hitachi Energy India Ltd", "Hitachi Energy India Ltd's", "Hitachi Energy India Ltd.", "Hitachi Energy India Ltd.'s", "Hitachi Energy India Private Lim", "Hitachi Energy India Private Lim's", "Hitachi Energy India Private Limited", "Hitachi Energy India Private Limited's", "Hitachi Energy India Pvt Ltd", "Hitachi Energy India Pvt Ltd's", "Hitachi Energy India Pvt. Ltd.", "Hitachi Energy India Pvt. Ltd.'s", "HitachiEI", "HitachiEI's", "HitachiEnergyIndia", "HitachiEnergyIndia's", "HitachiEnergyIndiaLim", "HitachiEnergyIndiaLim's", "HitachiEnergyIndiaLimited", "HitachiEnergyIndiaLimited's", "hitachienergyindia", "hitachienergyindia's", "hitachienergyindialimited"
        ]
    },

    "ABB India Limited": {
        "sbu": "India T&D",
        "aliases": [
"A&I", "A&I's", "A-I", "A.I", "A.I's", "ABB I", "ABB I's", "ABB India", "ABB India Lim", "ABB India Limited", "ABB India Ltd", "ABB India Ltd's", "ABB India Ltd.", "ABB India Ltd.'s", "ABB India Private Lim", "ABB India Private Lim's", "ABB India Private Limited", "ABB India Private Limited's", "ABB India Pvt Ltd", "ABB India Pvt Ltd's", "ABB India Pvt. Ltd.", "ABB India's", "ABBI", "ABBI's", "ABBIndia", "ABBIndia's", "ABBIndiaLim", "ABBIndiaLim's", "ABBIndiaLimited", "ABBIndiaLimited's", "AI", "AIL", "AIL's", "abbindia", "abbindia's", "abbindialimited", "abbindialimited's","Asea Brown Boveri","Aseabrownboveri"        ]
    },

    "Techno Electric & Engineering Company Limited": {
        "sbu": "India T&D",
        "aliases": [
"T&E&&&E", "T&E&&&E's", "T-E-&-E", "T-E-&-E's", "T.E.&.E", "T.E.&.E's", "TE&E", "TE&ECL", "TE&ECL's", "Techno E&E", "Techno Electric & Engineering Company", "Techno Electric & Engineering Company Lim", "Techno Electric & Engineering Company Lim's", "Techno Electric & Engineering Company Limited", "Techno Electric & Engineering Company Limited's", "Techno Electric & Engineering Company Ltd", "Techno Electric & Engineering Company Ltd.", "Techno Electric & Engineering Company Ltd.'s", "Techno Electric & Engineering Company Private Lim", "Techno Electric & Engineering Company Private Limited", "Techno Electric & Engineering Company Pvt Ltd", "Techno Electric & Engineering Company Pvt Ltd's", "Techno Electric & Engineering Company Pvt. Ltd.", "Techno Electric & Engineering Company's", "Techno Electric Engineering Company Lim", "Techno Electric Engineering Company Lim's", "Techno Electric Engineering Company Limited", "Techno Electric Engineering Company Limited's", "Techno Electric and Engineering Company Lim", "Techno Electric and Engineering Company Lim's", "Techno Electric and Engineering Company Limited", "Techno Electric and Engineering Company Limited's", "TechnoE&E", "TechnoE&E's", "TechnoElectric&EngineeringCompany", "TechnoElectric&EngineeringCompany's", "TechnoElectric&EngineeringCompanyLim", "TechnoElectric&EngineeringCompanyLim's", "TechnoElectric&EngineeringCompanyLimited", "TechnoElectric&EngineeringCompanyLimited's", "technoelectric&engineeringcompany", "technoelectric&engineeringcompany's", "technoelectric&engineeringcompanylimited", "technoelectric&engineeringcompanylimited's"        ]
    },

    "Jyoti Structures Limited": {
        "sbu": "India T&D",
        "aliases": [
 "J&S", "J&S's", "J-S", "J.S", "J.S's", "JS", "JSL", "JSL's", "Jyoti S", "Jyoti S's", "Jyoti Structures", "Jyoti Structures Lim", "Jyoti Structures Lim's", "Jyoti Structures Limited", "Jyoti Structures Limited's", "Jyoti Structures Ltd", "Jyoti Structures Ltd's", "Jyoti Structures Ltd.", "Jyoti Structures Ltd.'s", "Jyoti Structures Private Lim", "Jyoti Structures Private Limited", "Jyoti Structures Pvt Ltd", "Jyoti Structures Pvt Ltd's", "Jyoti Structures Pvt. Ltd.", "Jyoti Structures's", "JyotiS", "JyotiS's", "JyotiStructures", "JyotiStructures's", "JyotiStructuresLim", "JyotiStructuresLim's", "JyotiStructuresLimited", "JyotiStructuresLimited's", "jyotistructures", "jyotistructures's", "jyotistructureslimited", "jyotistructureslimited's"       ]
    },

    "Skipper Limited": {
        "sbu": "India T&D",
        "aliases": [
"SL", "Skipper", "Skipper Lim", "Skipper Lim's", "Skipper Limited", "Skipper Limited's", "Skipper Ltd", "Skipper Ltd's", "Skipper Ltd.", "Skipper Ltd.'s", "Skipper Private Lim", "Skipper Private Lim's", "Skipper Private Limited", "Skipper Private Limited's", "Skipper Pvt Ltd", "Skipper Pvt Ltd's", "Skipper Pvt. Ltd.", "Skipper Pvt. Ltd.'s", "Skipper's", "SkipperLim", "SkipperLim's", "SkipperLimited", "SkipperLimited's", "skipper", "skipper's", "skipperlimited", "skipperlimited's"        ]
    },

    "Hyosung T&D India Private Limited": {
        "sbu": "India T&D",
        "aliases": [
"H&T&I, "H&T&I's", "H-T-I", "H-T-I's", "H.T.I", "H.T.I's", "HTI", "HTIPL", "HTIPL's", "Hyosung T&D India Private", "Hyosung T&D India Private Lim", "Hyosung T&D India Private Lim's", "Hyosung T&D India Private Limited", "Hyosung T&D India Private Limited's", "Hyosung T&D India Private Ltd", "Hyosung T&D India Private Ltd.", "Hyosung T&D India Private Ltd.'s", "Hyosung T&D India Private Private Lim", "Hyosung T&D India Private Private Lim's", "Hyosung T&D India Private Private Limited", "Hyosung T&D India Private Private Limited's", "Hyosung T&D India Private Pvt Ltd", "Hyosung T&D India Private Pvt Ltd's", "Hyosung T&D India Private Pvt. Ltd.", "Hyosung T&D India Private's", "Hyosung TI", "Hyosung TandD India Private Lim", "Hyosung TandD India Private Lim's", "Hyosung TandD India Private Limited", "Hyosung TandD India Private Limited's", "HyosungT&DIndiaPrivate", "HyosungT&DIndiaPrivate's", "HyosungT&DIndiaPrivateLim", "HyosungT&DIndiaPrivateLim's", "HyosungT&DIndiaPrivateLimited", "HyosungT&DIndiaPrivateLimited's", "HyosungTI", "HyosungTI's", "hyosungt&dindiaprivate", "hyosungt&dindiaprivate's", "hyosungt&dindiaprivatelimited", "hyosungt&dindiaprivatelimited's"]
    },

    "NCC Limited": {
        "sbu": "India T&D",
        "aliases": [
 "NCC", "NCC Co.", "NCC Co.'s", "NCC Company", "NCC Company's", "NCC LLC", "NCC LLC's", "NCC Lim", "NCC Lim's", "NCC Limited", "NCC Limited's", "NCC Ltd", "NCC Ltd's", "NCC Ltd.", "NCC Ltd.'s", "NCC Private Lim", "NCC Private Lim's", "NCC Private Limited", "NCC Private Limited's", "NCC Pvt Ltd", "NCC Pvt Ltd's", "NCC Pvt. Ltd.", "NCC Pvt. Ltd.'s", "NCC's", "National Construction Company", "National Construction Company Co.", "National Construction Company Co.'s", "National Construction Company Company", "National Construction Company Company's", "National Construction Company LLC", "National Construction Company LLC's", "National Construction Company Lim", "National Construction Company Lim's", "National Construction Company Limited", "National Construction Company Limited's", "National Construction Company Ltd", "National Construction Company Ltd's", "National Construction Company Ltd.", "National Construction Company Ltd.'s", "National Construction Company Private Lim", "National Construction Company Private Lim's", "National Construction Company Private Limited", "National Construction Company Private Limited's", "National Construction Company Pvt Ltd", "National Construction Company Pvt Ltd's", "National Construction Company Pvt. Ltd.", "National Construction Company Pvt. Ltd.'s", "NationalConstructionCompany", "nationalconstructioncompany", "ncc", "ncc's"       ]
    },

    "Transrail Lighting Limited": {
        "sbu": "India T&D",
        "aliases": [
            "T&L", "T&L's", "T-L", "T-L's", "T.L", "T.L's", "TL", "TLL", "TLL's", "Transrail L", "Transrail L's", "Transrail Lighting", "Transrail Lighting Lim", "Transrail Lighting Limited", "Transrail Lighting Ltd", "Transrail Lighting Ltd's", "Transrail Lighting Ltd.", "Transrail Lighting Ltd.'s", "Transrail Lighting Private Lim", "Transrail Lighting Private Lim's", "Transrail Lighting Private Limited", "Transrail Lighting Private Limited's", "Transrail Lighting Pvt Ltd", "Transrail Lighting Pvt Ltd's", "Transrail Lighting Pvt. Ltd.", "Transrail Lighting Pvt. Ltd.'s", "TransrailL", "TransrailL's", "TransrailLighting", "TransrailLighting's", "TransrailLightingLim", "TransrailLightingLimited", "transraillighting", "transraillighting's", "transraillightinglimited", "transraillightinglimited's" ]
    },

    "Medha Servo Drives Limited": {
        "sbu": "India T&D",
        "aliases": [
            "M&S&D", "M&S&D's", "M-S-D", "M-S-D's", "M.S.D", "M.S.D's", "MSD", "MSD's", "MSDL", "Medha SD", "Medha Servo Drives", "Medha Servo Drives Lim", "Medha Servo Drives Lim's", "Medha Servo Drives Limited", "Medha Servo Drives Limited's", "Medha Servo Drives Ltd", "Medha Servo Drives Ltd's", "Medha Servo Drives Ltd.", "Medha Servo Drives Ltd.'s", "Medha Servo Drives Private Lim", "Medha Servo Drives Private Lim's", "Medha Servo Drives Private Limited", "Medha Servo Drives Private Limited's", "Medha Servo Drives Pvt Ltd", "Medha Servo Drives Pvt Ltd's", "Medha Servo Drives Pvt. Ltd.", "Medha Servo Drives's", "MedhaSD", "MedhaSD's", "MedhaServoDrives", "MedhaServoDrives's", "MedhaServoDrivesLim", "MedhaServoDrivesLim's", "MedhaServoDrivesLimited", "MedhaServoDrivesLimited's", "medhaservodrives", "medhaservodrives's", "medhaservodriveslimited", "medhaservodriveslimited's" ]
    },

    "Kernex Microsystems Private Limited": {
        "sbu": "India T&D",
        "aliases": [
            "K&M", "K-M", "K-M's", "K.M", "K.M's", "KM", "KMPL", "KMPL's", "Kernex M", "Kernex M's", "Kernex Microsystems Private", "Kernex Microsystems Private Lim", "Kernex Microsystems Private Lim's", "Kernex Microsystems Private Limited", "Kernex Microsystems Private Limited's", "Kernex Microsystems Private Ltd", "Kernex Microsystems Private Ltd's", "Kernex Microsystems Private Ltd.", "Kernex Microsystems Private Ltd.'s", "Kernex Microsystems Private Private Lim", "Kernex Microsystems Private Private Lim's", "Kernex Microsystems Private Private Limited", "Kernex Microsystems Private Private Limited's", "Kernex Microsystems Private Pvt Ltd", "Kernex Microsystems Private Pvt Ltd's", "Kernex Microsystems Private Pvt. Ltd.", "Kernex Microsystems Private Pvt. Ltd.'s", "KernexM", "KernexMicrosystemsPrivate", "KernexMicrosystemsPrivate's", "KernexMicrosystemsPrivateLim", "KernexMicrosystemsPrivateLim's", "KernexMicrosystemsPrivateLimited", "KernexMicrosystemsPrivateLimited's", "kernexmicrosystemsprivate", "kernexmicrosystemsprivate's", "kernexmicrosystemsprivatelimited", "kernexmicrosystemsprivatelimited's" ]
    },

    # --- International T&D ---
    "Larsen & Toubro Limited": {
        "sbu": "Intl T&D",
        "aliases": ["L&&&T", "L&&&T's", "L&T", "L&T's", "L&TL", "L&TL's", "L-&-T", "L-&-T's", "L.&.T", "L.&.T's", "Larsen & Toubro", "Larsen & Toubro Lim", "Larsen & Toubro Lim's", "Larsen & Toubro Limited", "Larsen & Toubro Limited's", "Larsen & Toubro Ltd", "Larsen & Toubro Ltd.", "Larsen & Toubro Ltd.'s", "Larsen & Toubro Private Lim", "Larsen & Toubro Private Lim's", "Larsen & Toubro Private Limited", "Larsen & Toubro Private Limited's", "Larsen & Toubro Pvt Ltd", "Larsen & Toubro Pvt Ltd's", "Larsen & Toubro Pvt. Ltd.", "Larsen &T", "Larsen &T's", "Larsen Toubro Lim", "Larsen Toubro Lim's", "Larsen Toubro Limited", "Larsen Toubro Limited's", "Larsen and Toubro Lim", "Larsen and Toubro Limited", "Larsen&T", "Larsen&T's", "Larsen&Toubro", "Larsen&Toubro's", "Larsen&ToubroLim", "Larsen&ToubroLim's", "Larsen&ToubroLimited", "Larsen&ToubroLimited's", "larsen&toubro", "larsen&toubro's", "larsen&toubrolimited" ]},
    "Kalpataru Projects International Limited": {
        "sbu": "Intl T&D",
        "aliases": ["K&P", "K-P", "K-P's", "K.P", "K.P's", "KP", "KPIL", "KPIL's", "Kalpataru P", "Kalpataru P's", "Kalpataru Projects International", "Kalpataru Projects International Lim", "Kalpataru Projects International Lim's", "Kalpataru Projects International Limited", "Kalpataru Projects International Limited's", "Kalpataru Projects International Ltd", "Kalpataru Projects International Ltd's", "Kalpataru Projects International Ltd.", "Kalpataru Projects International Ltd.'s", "Kalpataru Projects International Private Lim", "Kalpataru Projects International Private Lim's", "Kalpataru Projects International Private Limited", "Kalpataru Projects International Private Limited's", "Kalpataru Projects International Pvt Ltd", "Kalpataru Projects International Pvt Ltd's", "Kalpataru Projects International Pvt. Ltd.", "Kalpataru Projects International's", "KalpataruP", "KalpataruP's", "KalpataruProjectsInternational", "KalpataruProjectsInternational's", "KalpataruProjectsInternationalLim", "KalpataruProjectsInternationalLim's", "KalpataruProjectsInternationalLimited", "KalpataruProjectsInternationalLimited's", "kalpataruprojectsinternational", "kalpataruprojectsinternationallimited", "kalpataruprojectsinternationallimited's" ]},
    "Tata Projects Limited": {
        "sbu": "Intl T&D",
        "aliases": [ "T&P", "T&P's", "T-P", "T-P's", "T.P", "T.P's", "TP", "TPL", "TPL's", "Tata P", "Tata P's", "Tata Projects", "Tata Projects Lim", "Tata Projects Lim's", "Tata Projects Limited", "Tata Projects Limited's", "Tata Projects Ltd", "Tata Projects Ltd's", "Tata Projects Ltd.", "Tata Projects Ltd.'s", "Tata Projects Private Lim", "Tata Projects Private Lim's", "Tata Projects Private Limited", "Tata Projects Private Limited's", "Tata Projects Pvt Ltd", "Tata Projects Pvt. Ltd.", "Tata Projects Pvt. Ltd.'s", "Tata Projects's", "TataP", "TataP's", "TataProjects", "TataProjectsLim", "TataProjectsLim's", "TataProjectsLimited", "TataProjectsLimited's", "tataprojects", "tataprojects's", "tataprojectslimited", "tataprojectslimited's"]},
    "Sterlite Power Transmission Limited": {
        "sbu": "Intl T&D",
        "aliases": ["S&P&T", "S&P&T's", "S-P-T", "S-P-T's", "S.P.T", "SPT", "SPT's", "SPTL", "SPTL's", "Sterlite PT", "Sterlite PT's", "Sterlite Power Transmission", "Sterlite Power Transmission Lim", "Sterlite Power Transmission Lim's", "Sterlite Power Transmission Limited", "Sterlite Power Transmission Limited's", "Sterlite Power Transmission Ltd", "Sterlite Power Transmission Ltd's", "Sterlite Power Transmission Ltd.", "Sterlite Power Transmission Private Lim", "Sterlite Power Transmission Private Limited", "Sterlite Power Transmission Pvt Ltd", "Sterlite Power Transmission Pvt Ltd's", "Sterlite Power Transmission Pvt. Ltd.", "Sterlite Power Transmission Pvt. Ltd.'s", "Sterlite Power Transmission's", "SterlitePT", "SterlitePT's", "SterlitePowerTransmission", "SterlitePowerTransmission's", "SterlitePowerTransmissionLim", "SterlitePowerTransmissionLim's", "SterlitePowerTransmissionLimited", "SterlitePowerTransmissionLimited's", "sterlitepowertransmission", "sterlitepowertransmission's", "sterlitepowertransmissionlimited", "sterlitepowertransmissionlimited's" ]},
    "Bharat Heavy Electricals Limited": {
        "sbu": "Intl T&D",
        "aliases": ["B&H&E", "B-H-E", "B.H.E", "BHE", "BHE's", "BHEL", "BHEL's", "Bharat HE", "Bharat HE's", "Bharat Heavy Electricals", "Bharat Heavy Electricals Co.", "Bharat Heavy Electricals Co.'s", "Bharat Heavy Electricals Company", "Bharat Heavy Electricals Company's", "Bharat Heavy Electricals LLC", "Bharat Heavy Electricals Lim", "Bharat Heavy Electricals Lim's", "Bharat Heavy Electricals Limited", "Bharat Heavy Electricals Limited's", "Bharat Heavy Electricals Ltd", "Bharat Heavy Electricals Ltd's", "Bharat Heavy Electricals Ltd.", "Bharat Heavy Electricals Ltd.'s", "Bharat Heavy Electricals Private Lim", "Bharat Heavy Electricals Private Limited", "Bharat Heavy Electricals Pvt Ltd", "Bharat Heavy Electricals Pvt Ltd's", "Bharat Heavy Electricals Pvt. Ltd.", "Bharat Heavy Electricals's", "BharatHE", "BharatHE's", "BharatHeavyElectricals", "BharatHeavyElectricals's", "BharatHeavyElectricalsLim", "BharatHeavyElectricalsLim's", "BharatHeavyElectricalsLimited", "BharatHeavyElectricalsLimited's", "bharatheavyelectricals", "bharatheavyelectricals's", "bharatheavyelectricalslimited", "bharatheavyelectricalslimited's" ]},
    "Siemens Energy": {
        "sbu": "Intl T&D",
        "aliases": [
            "S&E", "S&E's", "S-E", "S.E", "S.E's", "SE", "Siemens E", "Siemens E's", "Siemens Energy", "Siemens Energy Co.", "Siemens Energy Company", "Siemens Energy Company's", "Siemens Energy LLC", "Siemens Energy Limited", "Siemens Energy Limited's", "Siemens Energy Ltd", "Siemens Energy Ltd's", "Siemens Energy Ltd.", "Siemens Energy Ltd.'s", "Siemens Energy Private Limited", "Siemens Energy Private Limited's", "Siemens Energy Pvt Ltd", "Siemens Energy Pvt Ltd's", "Siemens Energy Pvt. Ltd.", "Siemens Energy Pvt. Ltd.'s", "Siemens Energy's", "SiemensE", "SiemensE's", "SiemensEnergy", "SiemensEnergy's", "siemensenergy", "siemensenergy's"  ]
    },

    "Hitachi Energy": {
        "sbu": "Intl T&D",
        "aliases": [
H&E, "H&E's", "H-E", "H-E's", "H.E", "HE", "Hitachi E", "Hitachi E's", "Hitachi Energy", "Hitachi Energy Co.", "Hitachi Energy Co.'s", "Hitachi Energy Company", "Hitachi Energy Company's", "Hitachi Energy LLC", "Hitachi Energy Limited", "Hitachi Energy Limited's", "Hitachi Energy Ltd", "Hitachi Energy Ltd's", "Hitachi Energy Ltd.", "Hitachi Energy Ltd.'s", "Hitachi Energy Private Limited", "Hitachi Energy Private Limited's", "Hitachi Energy Pvt Ltd", "Hitachi Energy Pvt Ltd's", "Hitachi Energy Pvt. Ltd.", "Hitachi Energy Pvt. Ltd.'s", "Hitachi Energy's", "HitachiE", "HitachiE's", "HitachiEnergy", "HitachiEnergy's", "hitachienergy", "hitachienergy's"
        ]
    },

    "ABB": {
        "sbu": "Intl T&D",
        "aliases": [
   "ABB", "ABB Co.", "ABB Co.'s", "ABB Company", "ABB Company's", "ABB LLC", "ABB LLC's", "ABB Limited", "ABB Ltd", "ABB Ltd's", "ABB Ltd.", "ABB Ltd.'s", "ABB Private Limited", "ABB Private Limited's", "ABB Pvt Ltd", "ABB Pvt. Ltd.", "ABB Pvt. Ltd.'s", "ABB's", "Asea Brown Boveri", "Asea Brown Boveri Co.", "Asea Brown Boveri Company", "Asea Brown Boveri LLC", "Asea Brown Boveri Limited", "Asea Brown Boveri Ltd", "Asea Brown Boveri Ltd.", "Asea Brown Boveri Ltd.'s", "Asea Brown Boveri Private Limited", "Asea Brown Boveri Private Limited's", "Asea Brown Boveri Pvt Ltd", "Asea Brown Boveri Pvt Ltd's", "Asea Brown Boveri Pvt. Ltd.", "Asea Brown Boveri's", "AseaBrownBoveri", "AseaBrownBoveri's", "abb", "abb's", "aseabrownboveri", "aseabrownboveri's"     ]
    },

    "Hyundai Engineering & Construction Co. Ltd.": {
        "sbu": "Intl T&D",
        "aliases": [
            "H&E&&&C", "H&E&&&C's", "H-E-&-C", "H.E.&.C", "H.E.&.C's", "HE&C", "HE&C's", "HE&CC", "Hyundai E&C", "Hyundai E&C's", "Hyundai Engineering & Construction", "Hyundai Engineering & Construction Co.", "Hyundai Engineering & Construction Co.'s", "Hyundai Engineering & Construction Company", "Hyundai Engineering & Construction Company's", "Hyundai Engineering & Construction LLC", "Hyundai Engineering & Construction Limited", "Hyundai Engineering & Construction Limited's", "Hyundai Engineering & Construction Ltd", "Hyundai Engineering & Construction Ltd's", "Hyundai Engineering & Construction Ltd.", "Hyundai Engineering & Construction Ltd.'s", "Hyundai Engineering & Construction Private Limited", "Hyundai Engineering & Construction Pvt Ltd", "Hyundai Engineering & Construction Pvt Ltd's", "Hyundai Engineering & Construction Pvt. Ltd.", "Hyundai Engineering & Construction's", "Hyundai Engineering Construction Co.", "Hyundai Engineering Construction Co.'s", "Hyundai Engineering and Construction Co.", "Hyundai Engineering and Construction Co.'s", "HyundaiE&C", "HyundaiE&C's", "HyundaiEngineering&Construction", "HyundaiEngineering&ConstructionCo.", "hyundaiengineering&construction", "hyundaiengineering&constructionco.", "hyundaiengineering&constructionco.'s" ]
    },

    "Saudi Services For Electro Mechanic Works Company Limited": {
        "sbu": "Intl T&D",
        "aliases": [
            "S&E&M", "S&E&M's", "S-E-M", "S-E-M's", "S.E.M", "SEM", "SSFEMWCL", "SSFEMWCL's", "Saudi EM", "Saudi EM's", "Saudi Services For Electro Mechanic Works", "Saudi Services For Electro Mechanic Works Co.", "Saudi Services For Electro Mechanic Works Company", "Saudi Services For Electro Mechanic Works Company Lim", "Saudi Services For Electro Mechanic Works Company Limited", "Saudi Services For Electro Mechanic Works LLC", "Saudi Services For Electro Mechanic Works LLC's", "Saudi Services For Electro Mechanic Works Lim", "Saudi Services For Electro Mechanic Works Lim's", "Saudi Services For Electro Mechanic Works Limited", "Saudi Services For Electro Mechanic Works Limited's", "Saudi Services For Electro Mechanic Works Ltd", "Saudi Services For Electro Mechanic Works Ltd's", "Saudi Services For Electro Mechanic Works Ltd.", "Saudi Services For Electro Mechanic Works Ltd.'s", "Saudi Services For Electro Mechanic Works Private Lim", "Saudi Services For Electro Mechanic Works Private Limited", "Saudi Services For Electro Mechanic Works Pvt Ltd", "Saudi Services For Electro Mechanic Works Pvt. Ltd.", "Saudi Services For Electro Mechanic Works Pvt. Ltd.'s", "Saudi Services For Electro Mechanic Works's", "SaudiEM", "SaudiEM's", "SaudiServicesForElectroMechanicWorks", "SaudiServicesForElectroMechanicWorks's", "SaudiServicesForElectroMechanicWorksCompanyLim", "SaudiServicesForElectroMechanicWorksCompanyLim's", "SaudiServicesForElectroMechanicWorksCompanyLimited", "SaudiServicesForElectroMechanicWorksCompanyLimited's", "saudiservicesforelectromechanicworks", "saudiservicesforelectromechanicworks's", "saudiservicesforelectromechanicworkscompanylimited", "saudiservicesforelectromechanicworkscompanylimited's"  ]
    },

    "Emarat Aloula Contracting": {
        "sbu": "Intl T&D",
        "aliases": [
"E&A", "E&A's", "E-A", "E.A", "E.A's", "EA", "EAC", "Emarat A", "Emarat A's", "Emarat Aloula Contracting", "Emarat Aloula Contracting Co.", "Emarat Aloula Contracting Co.'s", "Emarat Aloula Contracting Company", "Emarat Aloula Contracting Company's", "Emarat Aloula Contracting LLC", "Emarat Aloula Contracting LLC's", "Emarat Aloula Contracting Limited", "Emarat Aloula Contracting Limited's", "Emarat Aloula Contracting Ltd", "Emarat Aloula Contracting Ltd's", "Emarat Aloula Contracting Ltd.", "Emarat Aloula Contracting Ltd.'s", "Emarat Aloula Contracting Private Limited", "Emarat Aloula Contracting Private Limited's", "Emarat Aloula Contracting Pvt Ltd", "Emarat Aloula Contracting Pvt. Ltd.", "Emarat Aloula Contracting Pvt. Ltd.'s", "Emarat Aloula Contracting's", "EmaratA", "EmaratAloulaContracting", "EmaratAloulaContracting's", "emarataloulacontracting", "emarataloulacontracting's"        ]
    },

    "Danway Electrical and Mechanical Engineering LLC": {
        "sbu": "Intl T&D",
        "aliases": ["D&E&A&M&E", "D&E&A&M&E's", "D-E-A-M-E", "D.E.A.M.E", "D.E.A.M.E's", "DEAME", "DEAME's", "DEAMEL", "Danway EAME", "Danway EAME's", "Danway Electrical & Mechanical Engineering LLC", "Danway Electrical & Mechanical Engineering LLC's", "Danway Electrical Mechanical Engineering LLC", "Danway Electrical Mechanical Engineering LLC's", "Danway Electrical and Mechanical Engineering", "Danway Electrical and Mechanical Engineering Co.", "Danway Electrical and Mechanical Engineering Company", "Danway Electrical and Mechanical Engineering Company's", "Danway Electrical and Mechanical Engineering LLC", "Danway Electrical and Mechanical Engineering Limited", "Danway Electrical and Mechanical Engineering Limited's", "Danway Electrical and Mechanical Engineering Ltd", "Danway Electrical and Mechanical Engineering Ltd's", "Danway Electrical and Mechanical Engineering Ltd.", "Danway Electrical and Mechanical Engineering Private Limited", "Danway Electrical and Mechanical Engineering Private Limited's", "Danway Electrical and Mechanical Engineering Pvt Ltd", "Danway Electrical and Mechanical Engineering Pvt Ltd's", "Danway Electrical and Mechanical Engineering Pvt. Ltd.", "Danway Electrical and Mechanical Engineering Pvt. Ltd.'s", "DanwayEAME", "DanwayEAME's", "DanwayElectricalandMechanicalEngineering", "DanwayElectricalandMechanicalEngineeringLLC", "DanwayElectricalandMechanicalEngineeringLLC's", "danwayelectricalandmechanicalengineering", "danwayelectricalandmechanicalengineering's", "danwayelectricalandmechanicalengineeringllc" ]
    },

    "Al Fanar Group": {
        "sbu": "Intl T&D",
        "aliases": [
"A&F", "A&F's", "A-F", "A-F's", "A.F", "A.F's", "AF", "AFG", "AFG's", "Al F", "Al Fanar Group", "Al Fanar Group Co.", "Al Fanar Group Company", "Al Fanar Group Company's", "Al Fanar Group LLC", "Al Fanar Group LLC's", "Al Fanar Group Limited", "Al Fanar Group Limited's", "Al Fanar Group Ltd", "Al Fanar Group Ltd's", "Al Fanar Group Ltd.", "Al Fanar Group Ltd.'s", "Al Fanar Group Private Limited", "Al Fanar Group Private Limited's", "Al Fanar Group Pvt Ltd", "Al Fanar Group Pvt. Ltd.", "Al Fanar Group Pvt. Ltd.'s", "Al Fanar Group's", "AlF", "AlFanarGroup", "AlFanarGroup's", "alfanargroup", "alfanargroup's"        ]
    },

    "Al Sharif Group Holding": {
        "sbu": "Intl T&D",
        "aliases": [
"A&S", "A-S", "A-S's", "A.S", "A.S's", "AS", "ASGH", "ASGH's", "Al S", "Al Sharif Group Holding", "Al Sharif Group Holding Co.", "Al Sharif Group Holding Company", "Al Sharif Group Holding Company's", "Al Sharif Group Holding LLC", "Al Sharif Group Holding LLC's", "Al Sharif Group Holding Limited", "Al Sharif Group Holding Limited's", "Al Sharif Group Holding Ltd", "Al Sharif Group Holding Ltd's", "Al Sharif Group Holding Ltd.", "Al Sharif Group Holding Ltd.'s", "Al Sharif Group Holding Private Limited", "Al Sharif Group Holding Private Limited's", "Al Sharif Group Holding Pvt Ltd", "Al Sharif Group Holding Pvt Ltd's", "Al Sharif Group Holding Pvt. Ltd.", "Al Sharif Group Holding Pvt. Ltd.'s", "Al Sharif Group Holding's", "AlS", "AlS's", "AlSharifGroupHolding", "AlSharifGroupHolding's", "alsharifgroupholding"        ]
    },

    # --- Civil ---
    "Larsen & Toubro Limited": {
        "sbu": "Civil",
        "aliases": ["L&&&T", "L&&&T's", "L&T", "L&T's", "L&TL", "L&TL's", "L-&-T", "L-&-T's", "L.&.T", "L.&.T's", "Larsen & Toubro", "Larsen & Toubro Co.", "Larsen & Toubro Co.'s", "Larsen & Toubro Company", "Larsen & Toubro Company's", "Larsen & Toubro LLC", "Larsen & Toubro Lim", "Larsen & Toubro Lim's", "Larsen & Toubro Limited", "Larsen & Toubro Limited's", "Larsen & Toubro Ltd", "Larsen & Toubro Ltd.", "Larsen & Toubro Ltd.'s", "Larsen & Toubro Private Lim", "Larsen & Toubro Private Limited", "Larsen & Toubro Pvt Ltd", "Larsen & Toubro Pvt. Ltd.", "Larsen &T", "Larsen &T's", "Larsen Toubro Lim", "Larsen Toubro Lim's", "Larsen Toubro Limited", "Larsen Toubro Limited's", "Larsen and Toubro Lim", "Larsen and Toubro Limited", "Larsen&T", "Larsen&T's", "Larsen&Toubro", "Larsen&Toubro's", "Larsen&ToubroLim", "Larsen&ToubroLim's", "Larsen&ToubroLimited", "Larsen&ToubroLimited's", "larsen&toubro", "larsen&toubro's", "larsen&toubrolimited"]},
    "Tata Projects Limited": {
        "sbu": "Civil",
        "aliases": ["T&P", "T&P's", "T-P", "T-P's", "T.P", "T.P's", "TP", "TPL", "TPL's", "Tata P", "Tata Projects", "Tata Projects Co.", "Tata Projects Company", "Tata Projects Company's", "Tata Projects LLC", "Tata Projects Lim", "Tata Projects Lim's", "Tata Projects Limited", "Tata Projects Limited's", "Tata Projects Ltd", "Tata Projects Ltd.", "Tata Projects Ltd.'s", "Tata Projects Private Lim", "Tata Projects Private Lim's", "Tata Projects Private Limited", "Tata Projects Private Limited's", "Tata Projects Pvt Ltd", "Tata Projects Pvt Ltd's", "Tata Projects Pvt. Ltd.", "Tata Projects Pvt. Ltd.'s", "Tata Projects's", "TataP", "TataP's", "TataProjects", "TataProjectsLim", "TataProjectsLimited", "tataprojects", "tataprojects's", "tataprojectslimited", "tataprojectslimited's"]},
    "Hindustan Construction Company Limited": {
        "sbu": "Civil",
        "aliases": ["H&C", "H&C's", "H-C", "H-C's", "H.C", "H.C's", "HC", "HCCL", "HCCL's", "Hindustan C", "Hindustan C's", "Hindustan Construction", "Hindustan Construction Co.", "Hindustan Construction Company", "Hindustan Construction Company Lim", "Hindustan Construction Company Lim's", "Hindustan Construction Company Limited", "Hindustan Construction Company Limited's", "Hindustan Construction LLC", "Hindustan Construction Lim", "Hindustan Construction Lim's", "Hindustan Construction Limited", "Hindustan Construction Limited's", "Hindustan Construction Ltd", "Hindustan Construction Ltd.", "Hindustan Construction Ltd.'s", "Hindustan Construction Private Lim", "Hindustan Construction Private Lim's", "Hindustan Construction Private Limited", "Hindustan Construction Private Limited's", "Hindustan Construction Pvt Ltd", "Hindustan Construction Pvt. Ltd.", "Hindustan Construction Pvt. Ltd.'s", "HindustanC", "HindustanC's", "HindustanConstruction", "HindustanConstruction's", "HindustanConstructionCompanyLim", "HindustanConstructionCompanyLim's", "HindustanConstructionCompanyLimited", "HindustanConstructionCompanyLimited's", "hindustanconstruction", "hindustanconstruction's", "hindustanconstructioncompanylimited"      ]
    },

    "Shapoorji Pallonji & Company Private Limited": {
        "sbu": "Civil",
        "aliases": ["S&P&&", "S&P&&'s", "S-P-&", "S.P.&", "SP&", "SP&CPL", "SP&CPL's", "Shapoorji P&", "Shapoorji Pallonji &  Private", "Shapoorji Pallonji &  Private Co.", "Shapoorji Pallonji &  Private Co.'s", "Shapoorji Pallonji &  Private Company", "Shapoorji Pallonji &  Private Company's", "Shapoorji Pallonji &  Private LLC", "Shapoorji Pallonji &  Private LLC's", "Shapoorji Pallonji &  Private Lim", "Shapoorji Pallonji &  Private Limited", "Shapoorji Pallonji &  Private Ltd", "Shapoorji Pallonji &  Private Ltd's", "Shapoorji Pallonji &  Private Ltd.", "Shapoorji Pallonji &  Private Ltd.'s", "Shapoorji Pallonji &  Private Private Lim", "Shapoorji Pallonji &  Private Private Limited", "Shapoorji Pallonji &  Private Pvt Ltd", "Shapoorji Pallonji &  Private Pvt Ltd's", "Shapoorji Pallonji &  Private Pvt. Ltd.", "Shapoorji Pallonji &  Private's", "Shapoorji Pallonji & Company Private Lim", "Shapoorji Pallonji & Company Private Lim's", "Shapoorji Pallonji & Company Private Limited", "Shapoorji Pallonji & Company Private Limited's", "Shapoorji Pallonji Company Private Lim", "Shapoorji Pallonji Company Private Lim's", "Shapoorji Pallonji Company Private Limited", "Shapoorji Pallonji Company Private Limited's", "Shapoorji Pallonji and Company Private Lim", "Shapoorji Pallonji and Company Private Lim's", "Shapoorji Pallonji and Company Private Limited", "Shapoorji Pallonji and Company Private Limited's", "ShapoorjiP&", "ShapoorjiP&'s", "ShapoorjiPallonji&CompanyPrivateLim", "ShapoorjiPallonji&CompanyPrivateLimited", "ShapoorjiPallonji&Private", "shapoorjipallonji&companyprivatelimited", "shapoorjipallonji&companyprivatelimited's", "shapoorjipallonji&private", "shapoorjipallonji&private's"        ]
    },

    "NCC Limited": {
        "sbu": "Civil",
        "aliases": ["NCC", "NCC Co.", "NCC Co.'s", "NCC Company", "NCC Company's", "NCC LLC", "NCC LLC's", "NCC Lim", "NCC Lim's", "NCC Limited", "NCC Limited's", "NCC Ltd", "NCC Ltd's", "NCC Ltd.", "NCC Ltd.'s", "NCC Private Lim", "NCC Private Lim's", "NCC Private Limited", "NCC Private Limited's", "NCC Pvt Ltd", "NCC Pvt Ltd's", "NCC Pvt. Ltd.", "NCC Pvt. Ltd.'s", "NCC's", "National Construction Company", "National Construction Company Co.", "National Construction Company Co.'s", "National Construction Company Company", "National Construction Company Company's", "National Construction Company LLC", "National Construction Company LLC's", "National Construction Company Lim", "National Construction Company Lim's", "National Construction Company Limited", "National Construction Company Limited's", "National Construction Company Ltd", "National Construction Company Ltd's", "National Construction Company Ltd.", "National Construction Company Ltd.'s", "National Construction Company Private Lim", "National Construction Company Private Lim's", "National Construction Company Private Limited", "National Construction Company Private Limited's", "National Construction Company Pvt Ltd", "National Construction Company Pvt Ltd's", "National Construction Company Pvt. Ltd.", "National Construction Company Pvt. Ltd.'s", "NationalConstructionCompany", "nationalconstructioncompany", "ncc", "ncc's"]},
    "Dilip Buildcon Limited": {
        "sbu": "Civil",
        "aliases": [
            "D&B", "D&B's", "D-B", "D-B's", "D.B", "DB", "DBL", "DBL's", "Dilip B", "Dilip B's", "Dilip Buildcon", "Dilip Buildcon Co.", "Dilip Buildcon Co.'s", "Dilip Buildcon Company", "Dilip Buildcon Company's", "Dilip Buildcon LLC", "Dilip Buildcon Lim", "Dilip Buildcon Limited", "Dilip Buildcon Ltd", "Dilip Buildcon Ltd's", "Dilip Buildcon Ltd.", "Dilip Buildcon Ltd.'s", "Dilip Buildcon Private Lim", "Dilip Buildcon Private Lim's", "Dilip Buildcon Private Limited", "Dilip Buildcon Private Limited's", "Dilip Buildcon Pvt Ltd", "Dilip Buildcon Pvt Ltd's", "Dilip Buildcon Pvt. Ltd.", "Dilip Buildcon Pvt. Ltd.'s", "Dilip Buildcon's", "DilipB", "DilipBuildcon", "DilipBuildcon's", "DilipBuildconLim", "DilipBuildconLimited", "dilipbuildcon", "dilipbuildconlimited", "dilipbuildconlimited's"  ]
    },

    "PNC Infratech Limited": {
        "sbu": "Civil",
        "aliases": ["P&I", "P-I", "P-I's", "P.I", "P.I's", "PI", "PIL", "PNC I", "PNC I's", "PNC Infratech", "PNC Infratech Co.", "PNC Infratech Co.'s", "PNC Infratech Company", "PNC Infratech Company's", "PNC Infratech LLC", "PNC Infratech LLC's", "PNC Infratech Lim", "PNC Infratech Lim's", "PNC Infratech Limited", "PNC Infratech Limited's", "PNC Infratech Ltd", "PNC Infratech Ltd's", "PNC Infratech Ltd.", "PNC Infratech Ltd.'s", "PNC Infratech Private Lim", "PNC Infratech Private Lim's", "PNC Infratech Private Limited", "PNC Infratech Private Limited's", "PNC Infratech Pvt Ltd", "PNC Infratech Pvt. Ltd.", "PNC Infratech's", "PNCI", "PNCI's", "PNCInfratech", "PNCInfratech's", "PNCInfratechLim", "PNCInfratechLim's", "PNCInfratechLimited", "PNCInfratechLimited's", "pncinfratech", "pncinfratechlimited"
        ]
    },

    "Simplex Infrastructures Limited": {
        "sbu": "Civil",
        "aliases": [
"S&I", "S&I's", "S-I", "S-I's", "S.I", "S.I's", "SI", "SIL", "Simplex I", "Simplex Infrastructures", "Simplex Infrastructures Co.", "Simplex Infrastructures Company", "Simplex Infrastructures Company's", "Simplex Infrastructures LLC", "Simplex Infrastructures Lim", "Simplex Infrastructures Lim's", "Simplex Infrastructures Limited", "Simplex Infrastructures Limited's", "Simplex Infrastructures Ltd", "Simplex Infrastructures Ltd's", "Simplex Infrastructures Ltd.", "Simplex Infrastructures Ltd.'s", "Simplex Infrastructures Private Lim", "Simplex Infrastructures Private Limited", "Simplex Infrastructures Pvt Ltd", "Simplex Infrastructures Pvt Ltd's", "Simplex Infrastructures Pvt. Ltd.", "Simplex Infrastructures Pvt. Ltd.'s", "Simplex Infrastructures's", "SimplexI", "SimplexI's", "SimplexInfrastructures", "SimplexInfrastructures's", "SimplexInfrastructuresLim", "SimplexInfrastructuresLim's", "SimplexInfrastructuresLimited", "SimplexInfrastructuresLimited's", "simplexinfrastructures", "simplexinfrastructureslimited", "simplexinfrastructureslimited's"        ]
    },

    "Ashoka Buildcon Limited": {
        "sbu": "Civil",
        "aliases": ["A&B", "A&B's", "A-B", "A-B's", "A.B", "A.B's", "AB", "ABL", "ABL's", "Ashoka B", "Ashoka B's", "Ashoka Buildcon", "Ashoka Buildcon Co.", "Ashoka Buildcon Co.'s", "Ashoka Buildcon Company", "Ashoka Buildcon Company's", "Ashoka Buildcon LLC", "Ashoka Buildcon Lim", "Ashoka Buildcon Lim's", "Ashoka Buildcon Limited", "Ashoka Buildcon Limited's", "Ashoka Buildcon Ltd", "Ashoka Buildcon Ltd's", "Ashoka Buildcon Ltd.", "Ashoka Buildcon Private Lim", "Ashoka Buildcon Private Lim's", "Ashoka Buildcon Private Limited", "Ashoka Buildcon Private Limited's", "Ashoka Buildcon Pvt Ltd", "Ashoka Buildcon Pvt. Ltd.", "Ashoka Buildcon's", "AshokaB", "AshokaBuildcon", "AshokaBuildcon's", "AshokaBuildconLim", "AshokaBuildconLimited", "ashokabuildcon", "ashokabuildcon's", "ashokabuildconlimited", "ashokabuildconlimited's" ]
    },

    "HG Infra Engineering Limited": {
        "sbu": "Civil",
        "aliases": ["H&E", "H&E's", "H-E", "H-E's", "H.E", "HE", "HG E", "HG Infra Engineering", "HG Infra Engineering Co.", "HG Infra Engineering Co.'s", "HG Infra Engineering Company", "HG Infra Engineering LLC", "HG Infra Engineering LLC's", "HG Infra Engineering Lim", "HG Infra Engineering Limited", "HG Infra Engineering Ltd", "HG Infra Engineering Ltd's", "HG Infra Engineering Ltd.", "HG Infra Engineering Ltd.'s", "HG Infra Engineering Private Lim", "HG Infra Engineering Private Lim's", "HG Infra Engineering Private Limited", "HG Infra Engineering Private Limited's", "HG Infra Engineering Pvt Ltd", "HG Infra Engineering Pvt Ltd's", "HG Infra Engineering Pvt. Ltd.", "HG Infra Engineering Pvt. Ltd.'s", "HG Infra Engineering's", "HGE", "HGInfraEngineering", "HGInfraEngineering's", "HGInfraEngineeringLim", "HGInfraEngineeringLim's", "HGInfraEngineeringLimited", "HGInfraEngineeringLimited's", "HIEL", "HIEL's", "hginfraengineering", "hginfraengineering's", "hginfraengineeringlimited"
        ]
    },

    "Ahluwalia Contracts (India) Limited": {
        "sbu": "Civil",
        "aliases": [
       "A&C", "A&C's", "A-C", "A-C's", "A.C", "A.C's", "AC", "AC(L", "Ahluwalia C", "Ahluwalia Contracts (India)", "Ahluwalia Contracts (India) Co.", "Ahluwalia Contracts (India) Co.'s", "Ahluwalia Contracts (India) Company", "Ahluwalia Contracts (India) LLC", "Ahluwalia Contracts (India) Lim", "Ahluwalia Contracts (India) Lim's", "Ahluwalia Contracts (India) Limited", "Ahluwalia Contracts (India) Limited's", "Ahluwalia Contracts (India) Ltd", "Ahluwalia Contracts (India) Ltd's", "Ahluwalia Contracts (India) Ltd.", "Ahluwalia Contracts (India) Ltd.'s", "Ahluwalia Contracts (India) Private Lim", "Ahluwalia Contracts (India) Private Lim's", "Ahluwalia Contracts (India) Private Limited", "Ahluwalia Contracts (India) Private Limited's", "Ahluwalia Contracts (India) Pvt Ltd", "Ahluwalia Contracts (India) Pvt Ltd's", "Ahluwalia Contracts (India) Pvt. Ltd.", "Ahluwalia Contracts (India) Pvt. Ltd.'s", "Ahluwalia Contracts (India)'s", "Ahluwalia Contracts India Lim", "Ahluwalia Contracts India Limited", "AhluwaliaC", "AhluwaliaContracts(India)", "AhluwaliaContracts(India)Lim", "AhluwaliaContracts(India)Lim's", "AhluwaliaContracts(India)Limited", "AhluwaliaContracts(India)Limited's", "ahluwalia contracts india limited", "ahluwaliacontracts(india)", "ahluwaliacontracts(india)'s", "ahluwaliacontracts(india)limited", "ahluwaliacontracts(india)limited's" ]
    },

    "AFCONS Infrastructure Limited": {
        "sbu": "Civil",
        "aliases": ["A&I", "A&I's", "A-I", "A.I", "A.I's", "AFCONS I", "AFCONS Infrastructure", "AFCONS Infrastructure Co.", "AFCONS Infrastructure Company", "AFCONS Infrastructure LLC", "AFCONS Infrastructure LLC's", "AFCONS Infrastructure Lim", "AFCONS Infrastructure Lim's", "AFCONS Infrastructure Limited", "AFCONS Infrastructure Limited's", "AFCONS Infrastructure Ltd", "AFCONS Infrastructure Ltd's", "AFCONS Infrastructure Ltd.", "AFCONS Infrastructure Ltd.'s", "AFCONS Infrastructure Private Lim", "AFCONS Infrastructure Private Lim's", "AFCONS Infrastructure Private Limited", "AFCONS Infrastructure Private Limited's", "AFCONS Infrastructure Pvt Ltd", "AFCONS Infrastructure Pvt Ltd's", "AFCONS Infrastructure Pvt. Ltd.", "AFCONS Infrastructure Pvt. Ltd.'s", "AFCONS Infrastructure's", "AFCONSI", "AFCONSI's", "AFCONSInfrastructure", "AFCONSInfrastructureLim", "AFCONSInfrastructureLim's", "AFCONSInfrastructureLimited", "AFCONSInfrastructureLimited's", "AI", "AIL", "AIL's", "afconsinfrastructure", "afconsinfrastructure's", "afconsinfrastructurelimited", "afconsinfrastructurelimited's"
        ]
    },

    "Kiran Infra Engineers Limited": {
        "sbu": "Civil",
        "aliases": ["K&E", "K-E", "K.E", "K.E's", "KE", "KIEL", "KIEL's", "Kiran E", "Kiran E's", "Kiran Infra Engineers", "Kiran Infra Engineers Co.", "Kiran Infra Engineers Company", "Kiran Infra Engineers Company's", "Kiran Infra Engineers LLC", "Kiran Infra Engineers LLC's", "Kiran Infra Engineers Lim", "Kiran Infra Engineers Lim's", "Kiran Infra Engineers Limited", "Kiran Infra Engineers Limited's", "Kiran Infra Engineers Ltd", "Kiran Infra Engineers Ltd's", "Kiran Infra Engineers Ltd.", "Kiran Infra Engineers Ltd.'s", "Kiran Infra Engineers Private Lim", "Kiran Infra Engineers Private Lim's", "Kiran Infra Engineers Private Limited", "Kiran Infra Engineers Private Limited's", "Kiran Infra Engineers Pvt Ltd", "Kiran Infra Engineers Pvt Ltd's", "Kiran Infra Engineers Pvt. Ltd.", "Kiran Infra Engineers Pvt. Ltd.'s", "Kiran Infra Engineers's", "KiranE", "KiranInfraEngineers", "KiranInfraEngineersLim", "KiranInfraEngineersLim's", "KiranInfraEngineersLimited", "KiranInfraEngineersLimited's", "kiraninfraengineers", "kiraninfraengineers's", "kiraninfraengineerslimited", "kiraninfraengineerslimited's"
        ]
    },

    "Dineshchandra R. Agrawal Infracon Private Limited": {
        "sbu": "Civil",
        "aliases": ["D&R&A", "D-R-A", "D-R-A's", "D.R.A", "D.R.A's", "DRA", "DRA's", "DRAIPL", "Dineshch&ra R. Agrawal Infracon Private Lim", "Dineshch&ra R. Agrawal Infracon Private Limited", "Dineshchandra R. Agrawal Infracon Private", "Dineshchandra R. Agrawal Infracon Private Co.", "Dineshchandra R. Agrawal Infracon Private Co.'s", "Dineshchandra R. Agrawal Infracon Private Company", "Dineshchandra R. Agrawal Infracon Private LLC", "Dineshchandra R. Agrawal Infracon Private LLC's", "Dineshchandra R. Agrawal Infracon Private Lim", "Dineshchandra R. Agrawal Infracon Private Lim's", "Dineshchandra R. Agrawal Infracon Private Limited", "Dineshchandra R. Agrawal Infracon Private Limited's", "Dineshchandra R. Agrawal Infracon Private Ltd", "Dineshchandra R. Agrawal Infracon Private Ltd's", "Dineshchandra R. Agrawal Infracon Private Ltd.", "Dineshchandra R. Agrawal Infracon Private Ltd.'s", "Dineshchandra R. Agrawal Infracon Private Private Lim", "Dineshchandra R. Agrawal Infracon Private Private Lim's", "Dineshchandra R. Agrawal Infracon Private Private Limited", "Dineshchandra R. Agrawal Infracon Private Private Limited's", "Dineshchandra R. Agrawal Infracon Private Pvt Ltd", "Dineshchandra R. Agrawal Infracon Private Pvt. Ltd.", "Dineshchandra R. Agrawal Infracon Private Pvt. Ltd.'s", "Dineshchandra R. Agrawal Infracon Private's", "Dineshchandra RA", "Dineshchandra RA's", "DineshchandraR.AgrawalInfraconPrivate", "DineshchandraR.AgrawalInfraconPrivate's", "DineshchandraR.AgrawalInfraconPrivateLim", "DineshchandraR.AgrawalInfraconPrivateLim's", "DineshchandraR.AgrawalInfraconPrivateLimited", "DineshchandraR.AgrawalInfraconPrivateLimited's", "DineshchandraRA", "DineshchandraRA's", "dineshchandrar.agrawalinfraconprivate", "dineshchandrar.agrawalinfraconprivatelimited"    ]
    },
    
    "Hyundai Engineering & Construction Co.": {
        "sbu": "Civil",
        "aliases": ["H&E&&&C", "H&E&&&C's", "H-E-&-C", "H.E.&.C", "H.E.&.C's", "HE&C", "HECC", "HECC's", "Hyundai E&C", "Hyundai E&C's", "Hyundai Engineering & Construction", "Hyundai Engineering & Construction Co.", "Hyundai Engineering & Construction Co.'s", "Hyundai Engineering & Construction Company", "Hyundai Engineering & Construction Company's", "Hyundai Engineering & Construction LLC", "Hyundai Engineering & Construction Limited", "Hyundai Engineering & Construction Limited's", "Hyundai Engineering & Construction Ltd", "Hyundai Engineering & Construction Ltd's", "Hyundai Engineering & Construction Ltd.", "Hyundai Engineering & Construction Ltd.'s", "Hyundai Engineering & Construction Private Limited", "Hyundai Engineering & Construction Pvt Ltd", "Hyundai Engineering & Construction Pvt Ltd's", "Hyundai Engineering & Construction Pvt. Ltd.", "Hyundai Engineering & Construction's", "Hyundai Engineering Construction Co.", "Hyundai Engineering Construction Co.'s", "Hyundai Engineering and Construction Co.", "Hyundai Engineering and Construction Co.'s", "HyundaiE&C", "HyundaiE&C's", "HyundaiEngineering&Construction", "HyundaiEngineering&ConstructionCo.", "hyundaiengineering&construction", "hyundaiengineering&constructionco.", "hyundaiengineering&constructionco.'s"]},
    
    # --- Transportation ---
    "IRCON International Limited": {
        "sbu": "Transportation",
        "aliases": [
            "IIL", "IIL's", "IRCON International", "IRCON International Co.", "IRCON International Co.'s", "IRCON International Company", "IRCON International Company's", "IRCON International LLC", "IRCON International LLC's", "IRCON International Lim", "IRCON International Lim's", "IRCON International Limited", "IRCON International Limited's", "IRCON International Ltd", "IRCON International Ltd's", "IRCON International Ltd.", "IRCON International Ltd.'s", "IRCON International Private Lim", "IRCON International Private Lim's", "IRCON International Private Limited", "IRCON International Private Limited's", "IRCON International Pvt Ltd", "IRCON International Pvt Ltd's", "IRCON International Pvt. Ltd.", "IRCON International's", "IRCONInternational", "IRCONInternational's", "IRCONInternationalLim", "IRCONInternationalLim's", "IRCONInternationalLimited", "IRCONInternationalLimited's", "irconinternational", "irconinternational's", "irconinternationallimited", "irconinternationallimited's" ]
    },
    "Tata Projects Limited": {
        "sbu": "Transportation",
        "aliases": ["T&P", "T&P's", "T-P", "T-P's", "T.P", "T.P's", "TP", "TPL", "TPL's", "Tata P", "Tata Projects", "Tata Projects Co.", "Tata Projects Company", "Tata Projects Company's", "Tata Projects LLC", "Tata Projects Lim", "Tata Projects Lim's", "Tata Projects Limited", "Tata Projects Limited's", "Tata Projects Ltd", "Tata Projects Ltd.", "Tata Projects Ltd.'s", "Tata Projects Private Lim", "Tata Projects Private Lim's", "Tata Projects Private Limited", "Tata Projects Private Limited's", "Tata Projects Pvt Ltd", "Tata Projects Pvt Ltd's", "Tata Projects Pvt. Ltd.", "Tata Projects Pvt. Ltd.'s", "Tata Projects's", "TataP", "TataP's", "TataProjects", "TataProjectsLim", "TataProjectsLimited", "tataprojects", "tataprojects's", "tataprojectslimited", "tataprojectslimited's"]},
    

    "NCC Limited": {
        "sbu": "Transportation",
        "aliases": ["NCC", "NCC Co.", "NCC Co.'s", "NCC Company", "NCC Company's", "NCC LLC", "NCC LLC's", "NCC Lim", "NCC Lim's", "NCC Limited", "NCC Limited's", "NCC Ltd", "NCC Ltd's", "NCC Ltd.", "NCC Ltd.'s", "NCC Private Lim", "NCC Private Lim's", "NCC Private Limited", "NCC Private Limited's", "NCC Pvt Ltd", "NCC Pvt Ltd's", "NCC Pvt. Ltd.", "NCC Pvt. Ltd.'s", "NCC's", "National Construction Company", "National Construction Company Co.", "National Construction Company Co.'s", "National Construction Company Company", "National Construction Company Company's", "National Construction Company LLC", "National Construction Company LLC's", "National Construction Company Lim", "National Construction Company Lim's", "National Construction Company Limited", "National Construction Company Limited's", "National Construction Company Ltd", "National Construction Company Ltd's", "National Construction Company Ltd.", "National Construction Company Ltd.'s", "National Construction Company Private Lim", "National Construction Company Private Lim's", "National Construction Company Private Limited", "National Construction Company Private Limited's", "National Construction Company Pvt Ltd", "National Construction Company Pvt Ltd's", "National Construction Company Pvt. Ltd.", "National Construction Company Pvt. Ltd.'s", "NationalConstructionCompany", "nationalconstructioncompany", "ncc", "ncc's"]},
    "Larsen & Toubro Limited": {
        "sbu": "Transportation",
        "aliases": ["L&&&T", "L&&&T's", "L&T", "L&T's", "L&TL", "L&TL's", "L-&-T", "L-&-T's", "L.&.T", "L.&.T's", "Larsen & Toubro", "Larsen & Toubro Co.", "Larsen & Toubro Co.'s", "Larsen & Toubro Company", "Larsen & Toubro Company's", "Larsen & Toubro LLC", "Larsen & Toubro Lim", "Larsen & Toubro Lim's", "Larsen & Toubro Limited", "Larsen & Toubro Limited's", "Larsen & Toubro Ltd", "Larsen & Toubro Ltd.", "Larsen & Toubro Ltd.'s", "Larsen & Toubro Private Lim", "Larsen & Toubro Private Limited", "Larsen & Toubro Pvt Ltd", "Larsen & Toubro Pvt. Ltd.", "Larsen &T", "Larsen &T's", "Larsen Toubro Lim", "Larsen Toubro Lim's", "Larsen Toubro Limited", "Larsen Toubro Limited's", "Larsen and Toubro Lim", "Larsen and Toubro Limited", "Larsen&T", "Larsen&T's", "Larsen&Toubro", "Larsen&Toubro's", "Larsen&ToubroLim", "Larsen&ToubroLim's", "Larsen&ToubroLimited", "Larsen&ToubroLimited's", "larsen&toubro", "larsen&toubro's", "larsen&toubrolimited"]},
    "Hindustan Construction Company Limited": {
        "sbu": "Transportation",
        "aliases": ["H&C", "H&C's", "H-C", "H-C's", "H.C", "H.C's", "HC", "HCCL", "HCCL's", "Hindustan C", "Hindustan C's", "Hindustan Construction", "Hindustan Construction Co.", "Hindustan Construction Company", "Hindustan Construction Company Lim", "Hindustan Construction Company Lim's", "Hindustan Construction Company Limited", "Hindustan Construction Company Limited's", "Hindustan Construction LLC", "Hindustan Construction Lim", "Hindustan Construction Lim's", "Hindustan Construction Limited", "Hindustan Construction Limited's", "Hindustan Construction Ltd", "Hindustan Construction Ltd.", "Hindustan Construction Ltd.'s", "Hindustan Construction Private Lim", "Hindustan Construction Private Lim's", "Hindustan Construction Private Limited", "Hindustan Construction Private Limited's", "Hindustan Construction Pvt Ltd", "Hindustan Construction Pvt. Ltd.", "Hindustan Construction Pvt. Ltd.'s", "HindustanC", "HindustanC's", "HindustanConstruction", "HindustanConstruction's", "HindustanConstructionCompanyLim", "HindustanConstructionCompanyLim's", "HindustanConstructionCompanyLimited", "HindustanConstructionCompanyLimited's", "hindustanconstruction", "hindustanconstruction's", "hindustanconstructioncompanylimited"]},
    "Texmaco Rail & Engineering Limited": {
        "sbu": "Transportation",
        "aliases": [
           "T&R&&&E", "T&R&&&E's", "T-R-&-E", "T.R.&.E", "T.R.&.E's", "TR&E", "TR&E's", "TR&EL", "TR&EL's", "Texmaco R&E", "Texmaco R&E's", "Texmaco Rail & Engineering", "Texmaco Rail & Engineering Co.", "Texmaco Rail & Engineering Company", "Texmaco Rail & Engineering Company's", "Texmaco Rail & Engineering LLC", "Texmaco Rail & Engineering LLC's", "Texmaco Rail & Engineering Lim", "Texmaco Rail & Engineering Lim's", "Texmaco Rail & Engineering Limited", "Texmaco Rail & Engineering Limited's", "Texmaco Rail & Engineering Ltd", "Texmaco Rail & Engineering Ltd.", "Texmaco Rail & Engineering Private Lim", "Texmaco Rail & Engineering Private Lim's", "Texmaco Rail & Engineering Private Limited", "Texmaco Rail & Engineering Private Limited's", "Texmaco Rail & Engineering Pvt Ltd", "Texmaco Rail & Engineering Pvt. Ltd.", "Texmaco Rail & Engineering Pvt. Ltd.'s", "Texmaco Rail & Engineering's", "Texmaco Rail Engineering Lim", "Texmaco Rail Engineering Lim's", "Texmaco Rail Engineering Limited", "Texmaco Rail Engineering Limited's", "Texmaco Rail and Engineering Lim", "Texmaco Rail and Engineering Limited", "TexmacoR&E", "TexmacoR&E's", "TexmacoRail&Engineering", "TexmacoRail&EngineeringLim", "TexmacoRail&EngineeringLim's", "TexmacoRail&EngineeringLimited", "TexmacoRail&EngineeringLimited's", "texmacorail&engineering", "texmacorail&engineering's", "texmacorail&engineeringlimited" ]
    },

    "Simplex Infrastructures Limited": {
        "sbu": "Transportation",
        "aliases": ["S&I", "S&I's", "S-I", "S-I's", "S.I", "S.I's", "SI", "SIL", "Simplex I", "Simplex Infrastructures", "Simplex Infrastructures Co.", "Simplex Infrastructures Company", "Simplex Infrastructures Company's", "Simplex Infrastructures LLC", "Simplex Infrastructures Lim", "Simplex Infrastructures Lim's", "Simplex Infrastructures Limited", "Simplex Infrastructures Limited's", "Simplex Infrastructures Ltd", "Simplex Infrastructures Ltd's", "Simplex Infrastructures Ltd.", "Simplex Infrastructures Ltd.'s", "Simplex Infrastructures Private Lim", "Simplex Infrastructures Private Limited", "Simplex Infrastructures Pvt Ltd", "Simplex Infrastructures Pvt Ltd's", "Simplex Infrastructures Pvt. Ltd.", "Simplex Infrastructures Pvt. Ltd.'s", "Simplex Infrastructures's", "SimplexI", "SimplexI's", "SimplexInfrastructures", "SimplexInfrastructures's", "SimplexInfrastructuresLim", "SimplexInfrastructuresLim's", "SimplexInfrastructuresLimited", "SimplexInfrastructuresLimited's", "simplexinfrastructures", "simplexinfrastructureslimited", "simplexinfrastructureslimited's"]},
    "Dilip Buildcon Limited": {
        "sbu": "Transportation",
        "aliases": ["D&B", "D&B's", "D-B", "D-B's", "D.B", "DB", "DBL", "DBL's", "Dilip B", "Dilip B's", "Dilip Buildcon", "Dilip Buildcon Co.", "Dilip Buildcon Co.'s", "Dilip Buildcon Company", "Dilip Buildcon Company's", "Dilip Buildcon LLC", "Dilip Buildcon Lim", "Dilip Buildcon Limited", "Dilip Buildcon Ltd", "Dilip Buildcon Ltd's", "Dilip Buildcon Ltd.", "Dilip Buildcon Ltd.'s", "Dilip Buildcon Private Lim", "Dilip Buildcon Private Lim's", "Dilip Buildcon Private Limited", "Dilip Buildcon Private Limited's", "Dilip Buildcon Pvt Ltd", "Dilip Buildcon Pvt Ltd's", "Dilip Buildcon Pvt. Ltd.", "Dilip Buildcon Pvt. Ltd.'s", "Dilip Buildcon's", "DilipB", "DilipBuildcon", "DilipBuildcon's", "DilipBuildconLim", "DilipBuildconLimited", "dilipbuildcon", "dilipbuildconlimited", "dilipbuildconlimited's"]},
    "PNC Infratech Limited": {
        "sbu": "Transportation",
        "aliases": ["P&I", "P-I", "P-I's", "P.I", "P.I's", "PI", "PIL", "PNC I", "PNC I's", "PNC Infratech", "PNC Infratech Co.", "PNC Infratech Co.'s", "PNC Infratech Company", "PNC Infratech Company's", "PNC Infratech LLC", "PNC Infratech LLC's", "PNC Infratech Lim", "PNC Infratech Lim's", "PNC Infratech Limited", "PNC Infratech Limited's", "PNC Infratech Ltd", "PNC Infratech Ltd's", "PNC Infratech Ltd.", "PNC Infratech Ltd.'s", "PNC Infratech Private Lim", "PNC Infratech Private Lim's", "PNC Infratech Private Limited", "PNC Infratech Private Limited's", "PNC Infratech Pvt Ltd", "PNC Infratech Pvt. Ltd.", "PNC Infratech's", "PNCI", "PNCI's", "PNCInfratech", "PNCInfratech's", "PNCInfratechLim", "PNCInfratechLim's", "PNCInfratechLimited", "PNCInfratechLimited's", "pncinfratech", "pncinfratechlimited"]},
    "AFCONS Infrastructure Limited": {
        "sbu": "Transportation",
        "aliases": ["A&I", "A&I's", "A-I", "A.I", "A.I's", "AFCONS I", "AFCONS Infrastructure", "AFCONS Infrastructure Co.", "AFCONS Infrastructure Company", "AFCONS Infrastructure LLC", "AFCONS Infrastructure LLC's", "AFCONS Infrastructure Lim", "AFCONS Infrastructure Lim's", "AFCONS Infrastructure Limited", "AFCONS Infrastructure Limited's", "AFCONS Infrastructure Ltd", "AFCONS Infrastructure Ltd's", "AFCONS Infrastructure Ltd.", "AFCONS Infrastructure Ltd.'s", "AFCONS Infrastructure Private Lim", "AFCONS Infrastructure Private Lim's", "AFCONS Infrastructure Private Limited", "AFCONS Infrastructure Private Limited's", "AFCONS Infrastructure Pvt Ltd", "AFCONS Infrastructure Pvt Ltd's", "AFCONS Infrastructure Pvt. Ltd.", "AFCONS Infrastructure Pvt. Ltd.'s", "AFCONS Infrastructure's", "AFCONSI", "AFCONSI's", "AFCONSInfrastructure", "AFCONSInfrastructureLim", "AFCONSInfrastructureLim's", "AFCONSInfrastructureLimited", "AFCONSInfrastructureLimited's", "AI", "AIL", "AIL's", "afconsinfrastructure", "afconsinfrastructure's", "afconsinfrastructurelimited", "afconsinfrastructurelimited's"]},
    "HG Infra Engineering Limited": {
        "sbu": "Transportation",
        "aliases": ["H&E", "H&E's", "H-E", "H-E's", "H.E", "HE", "HG E", "HG Infra Engineering", "HG Infra Engineering Co.", "HG Infra Engineering Co.'s", "HG Infra Engineering Company", "HG Infra Engineering LLC", "HG Infra Engineering LLC's", "HG Infra Engineering Lim", "HG Infra Engineering Limited", "HG Infra Engineering Ltd", "HG Infra Engineering Ltd's", "HG Infra Engineering Ltd.", "HG Infra Engineering Ltd.'s", "HG Infra Engineering Private Lim", "HG Infra Engineering Private Lim's", "HG Infra Engineering Private Limited", "HG Infra Engineering Private Limited's", "HG Infra Engineering Pvt Ltd", "HG Infra Engineering Pvt Ltd's", "HG Infra Engineering Pvt. Ltd.", "HG Infra Engineering Pvt. Ltd.'s", "HG Infra Engineering's", "HGE", "HGInfraEngineering", "HGInfraEngineering's", "HGInfraEngineeringLim", "HGInfraEngineeringLim's", "HGInfraEngineeringLimited", "HGInfraEngineeringLimited's", "HIEL", "HIEL's", "hginfraengineering", "hginfraengineering's", "hginfraengineeringlimited"]},
    "Ashoka Buildcon Limited": {
        "sbu": "Transportation",
        "aliases": ["A&B", "A&B's", "A-B", "A-B's", "A.B", "A.B's", "AB", "ABL", "ABL's", "Ashoka B", "Ashoka B's", "Ashoka Buildcon", "Ashoka Buildcon Co.", "Ashoka Buildcon Co.'s", "Ashoka Buildcon Company", "Ashoka Buildcon Company's", "Ashoka Buildcon LLC", "Ashoka Buildcon Lim", "Ashoka Buildcon Lim's", "Ashoka Buildcon Limited", "Ashoka Buildcon Limited's", "Ashoka Buildcon Ltd", "Ashoka Buildcon Ltd's", "Ashoka Buildcon Ltd.", "Ashoka Buildcon Private Lim", "Ashoka Buildcon Private Lim's", "Ashoka Buildcon Private Limited", "Ashoka Buildcon Private Limited's", "Ashoka Buildcon Pvt Ltd", "Ashoka Buildcon Pvt. Ltd.", "Ashoka Buildcon's", "AshokaB", "AshokaBuildcon", "AshokaBuildcon's", "AshokaBuildconLim", "AshokaBuildconLimited", "ashokabuildcon", "ashokabuildcon's", "ashokabuildconlimited", "ashokabuildconlimited's"]},
    "Ahluwalia Contracts (India) Limited": {
        "sbu": "Transportation",
        "aliases": ["A&C", "A&C's", "A-C", "A-C's", "A.C", "A.C's", "AC", "AC(L", "Ahluwalia C", "Ahluwalia Contracts (India)", "Ahluwalia Contracts (India) Co.", "Ahluwalia Contracts (India) Co.'s", "Ahluwalia Contracts (India) Company", "Ahluwalia Contracts (India) LLC", "Ahluwalia Contracts (India) Lim", "Ahluwalia Contracts (India) Lim's", "Ahluwalia Contracts (India) Limited", "Ahluwalia Contracts (India) Limited's", "Ahluwalia Contracts (India) Ltd", "Ahluwalia Contracts (India) Ltd's", "Ahluwalia Contracts (India) Ltd.", "Ahluwalia Contracts (India) Ltd.'s", "Ahluwalia Contracts (India) Private Lim", "Ahluwalia Contracts (India) Private Lim's", "Ahluwalia Contracts (India) Private Limited", "Ahluwalia Contracts (India) Private Limited's", "Ahluwalia Contracts (India) Pvt Ltd", "Ahluwalia Contracts (India) Pvt Ltd's", "Ahluwalia Contracts (India) Pvt. Ltd.", "Ahluwalia Contracts (India) Pvt. Ltd.'s", "Ahluwalia Contracts (India)'s", "Ahluwalia Contracts India Lim", "Ahluwalia Contracts India Limited", "AhluwaliaC", "AhluwaliaContracts(India)", "AhluwaliaContracts(India)Lim", "AhluwaliaContracts(India)Lim's", "AhluwaliaContracts(India)Limited", "AhluwaliaContracts(India)Limited's", "ahluwalia contracts india limited", "ahluwaliacontracts(india)", "ahluwaliacontracts(india)'s", "ahluwaliacontracts(india)limited", "ahluwaliacontracts(india)limited's"]},
    "Shapoorji Pallonji & Company Private Limited": {
        "sbu": "Transportation",
        "aliases": ["S&P&&", "S&P&&'s", "S-P-&", "S.P.&", "SP&", "SPCPL", "SPCPL's", "Shapoorji P&", "Shapoorji Pallonji &  Private", "Shapoorji Pallonji &  Private Co.", "Shapoorji Pallonji &  Private Co.'s", "Shapoorji Pallonji &  Private Company", "Shapoorji Pallonji &  Private Company's", "Shapoorji Pallonji &  Private LLC", "Shapoorji Pallonji &  Private LLC's", "Shapoorji Pallonji &  Private Lim", "Shapoorji Pallonji &  Private Limited", "Shapoorji Pallonji &  Private Ltd", "Shapoorji Pallonji &  Private Ltd's", "Shapoorji Pallonji &  Private Ltd.", "Shapoorji Pallonji &  Private Ltd.'s", "Shapoorji Pallonji &  Private Private Lim", "Shapoorji Pallonji &  Private Private Limited", "Shapoorji Pallonji &  Private Pvt Ltd", "Shapoorji Pallonji &  Private Pvt Ltd's", "Shapoorji Pallonji &  Private Pvt. Ltd.", "Shapoorji Pallonji &  Private's", "Shapoorji Pallonji & Company Private Lim", "Shapoorji Pallonji & Company Private Lim's", "Shapoorji Pallonji & Company Private Limited", "Shapoorji Pallonji & Company Private Limited's", "Shapoorji Pallonji Company Private Lim", "Shapoorji Pallonji Company Private Lim's", "Shapoorji Pallonji Company Private Limited", "Shapoorji Pallonji Company Private Limited's", "Shapoorji Pallonji and Company Private Lim", "Shapoorji Pallonji and Company Private Lim's", "Shapoorji Pallonji and Company Private Limited", "Shapoorji Pallonji and Company Private Limited's", "ShapoorjiP&", "ShapoorjiP&'s", "ShapoorjiPallonji&CompanyPrivateLim", "ShapoorjiPallonji&CompanyPrivateLimited", "ShapoorjiPallonji&Private", "shapoorjipallonji&companyprivatelimited", "shapoorjipallonji&companyprivatelimited's", "shapoorjipallonji&private", "shapoorjipallonji&private's"]},
    "Bharat Heavy Electricals Limited": {
        "sbu": "Transportation",
        "aliases": ["B&H&E", "B-H-E", "B.H.E", "BHE", "BHE's", "BHEL", "BHEL's", "Bharat HE", "Bharat HE's", "Bharat Heavy Electricals", "Bharat Heavy Electricals Co.", "Bharat Heavy Electricals Co.'s", "Bharat Heavy Electricals Company", "Bharat Heavy Electricals Company's", "Bharat Heavy Electricals LLC", "Bharat Heavy Electricals Lim", "Bharat Heavy Electricals Lim's", "Bharat Heavy Electricals Limited", "Bharat Heavy Electricals Limited's", "Bharat Heavy Electricals Ltd", "Bharat Heavy Electricals Ltd's", "Bharat Heavy Electricals Ltd.", "Bharat Heavy Electricals Ltd.'s", "Bharat Heavy Electricals Private Lim", "Bharat Heavy Electricals Private Limited", "Bharat Heavy Electricals Pvt Ltd", "Bharat Heavy Electricals Pvt Ltd's", "Bharat Heavy Electricals Pvt. Ltd.", "Bharat Heavy Electricals's", "BharatHE", "BharatHE's", "BharatHeavyElectricals", "BharatHeavyElectricals's", "BharatHeavyElectricalsLim", "BharatHeavyElectricalsLim's", "BharatHeavyElectricalsLimited", "BharatHeavyElectricalsLimited's", "bharatheavyelectricals", "bharatheavyelectricals's", "bharatheavyelectricalslimited", "bharatheavyelectricalslimited's"]},
    "Medha Servo Drives Limited": {
        "sbu": "Transportation",
        "aliases": ["M&S&D", "M&S&D's", "M-S-D", "M.S.D", "MSD", "MSD's", "MSDL", "MSDL's", "Medha SD", "Medha SD's", "Medha Servo Drives", "Medha Servo Drives Co.", "Medha Servo Drives Company", "Medha Servo Drives Company's", "Medha Servo Drives LLC", "Medha Servo Drives LLC's", "Medha Servo Drives Lim", "Medha Servo Drives Lim's", "Medha Servo Drives Limited", "Medha Servo Drives Limited's", "Medha Servo Drives Ltd", "Medha Servo Drives Ltd's", "Medha Servo Drives Ltd.", "Medha Servo Drives Private Lim", "Medha Servo Drives Private Lim's", "Medha Servo Drives Private Limited", "Medha Servo Drives Private Limited's", "Medha Servo Drives Pvt Ltd", "Medha Servo Drives Pvt Ltd's", "Medha Servo Drives Pvt. Ltd.", "Medha Servo Drives Pvt. Ltd.'s", "Medha Servo Drives's", "MedhaSD", "MedhaSD's", "MedhaServoDrives", "MedhaServoDrivesLim", "MedhaServoDrivesLim's", "MedhaServoDrivesLimited", "MedhaServoDrivesLimited's", "medhaservodrives", "medhaservodrives's", "medhaservodriveslimited"]},
    "Hyundai Engineering & Construction Co.": {
        "sbu": "Transportation",
        "aliases": ["H&E&&&C", "H&E&&&C's", "H-E-&-C", "H.E.&.C", "H.E.&.C's", "HE&C", "HECC", "HECC's", "Hyundai E&C", "Hyundai E&C's", "Hyundai Engineering & Construction", "Hyundai Engineering & Construction Co.", "Hyundai Engineering & Construction Co.'s", "Hyundai Engineering & Construction Company", "Hyundai Engineering & Construction Company's", "Hyundai Engineering & Construction LLC", "Hyundai Engineering & Construction Limited", "Hyundai Engineering & Construction Limited's", "Hyundai Engineering & Construction Ltd", "Hyundai Engineering & Construction Ltd's", "Hyundai Engineering & Construction Ltd.", "Hyundai Engineering & Construction Ltd.'s", "Hyundai Engineering & Construction Private Limited", "Hyundai Engineering & Construction Pvt Ltd", "Hyundai Engineering & Construction Pvt Ltd's", "Hyundai Engineering & Construction Pvt. Ltd.", "Hyundai Engineering & Construction's", "Hyundai Engineering Construction Co.", "Hyundai Engineering Construction Co.'s", "Hyundai Engineering and Construction Co.", "Hyundai Engineering and Construction Co.'s", "HyundaiE&C", "HyundaiE&C's", "HyundaiEngineering&Construction", "HyundaiEngineering&ConstructionCo.", "hyundaiengineering&construction", "hyundaiengineering&constructionco.", "hyundaiengineering&constructionco.'s"]},
    "Rail Vikas Nigam Limited": {
        "sbu": "Transportation",
        "aliases": [
            "R&V&N", "R&V&N's", "R-V-N", "R.V.N", "RVN", "RVN's", "RVNL", "Rail VN", "Rail VN's", "Rail Vikas Nigam", "Rail Vikas Nigam Co.", "Rail Vikas Nigam Co.'s", "Rail Vikas Nigam Company", "Rail Vikas Nigam Company's", "Rail Vikas Nigam LLC", "Rail Vikas Nigam LLC's", "Rail Vikas Nigam Lim", "Rail Vikas Nigam Lim's", "Rail Vikas Nigam Limited", "Rail Vikas Nigam Limited's", "Rail Vikas Nigam Ltd", "Rail Vikas Nigam Ltd's", "Rail Vikas Nigam Ltd.", "Rail Vikas Nigam Ltd.'s", "Rail Vikas Nigam Private Lim", "Rail Vikas Nigam Private Lim's", "Rail Vikas Nigam Private Limited", "Rail Vikas Nigam Private Limited's", "Rail Vikas Nigam Pvt Ltd", "Rail Vikas Nigam Pvt Ltd's", "Rail Vikas Nigam Pvt. Ltd.", "Rail Vikas Nigam Pvt. Ltd.'s", "Rail Vikas Nigam's", "RailVN", "RailVN's", "RailVikasNigam", "RailVikasNigamLim", "RailVikasNigamLim's", "RailVikasNigamLimited", "RailVikasNigamLimited's", "railvikasnigam", "railvikasnigamlimited"
        ]
    },

    # --- Renewables ---
    "Tata Power Solar Systems Limited": {
        "sbu": "Renewables",
        "aliases": [
"T&P&S&S", "T&P&S&S's", "T-P-S-S", "T-P-S-S's", "T.P.S.S", "T.P.S.S's", "TPSS", "TPSS's", "TPSSL", "TPSSL's", "Tata PSS", "Tata PSS's", "Tata Power Solar Systems", "Tata Power Solar Systems Co.", "Tata Power Solar Systems Company", "Tata Power Solar Systems Company's", "Tata Power Solar Systems LLC", "Tata Power Solar Systems Lim", "Tata Power Solar Systems Limited", "Tata Power Solar Systems Ltd", "Tata Power Solar Systems Ltd.", "Tata Power Solar Systems PLC", "Tata Power Solar Systems PLC's", "Tata Power Solar Systems Private Lim", "Tata Power Solar Systems Private Lim's", "Tata Power Solar Systems Private Limited", "Tata Power Solar Systems Private Limited's", "Tata Power Solar Systems Pvt Ltd", "Tata Power Solar Systems Pvt Ltd's", "Tata Power Solar Systems Pvt. Ltd.", "Tata Power Solar Systems Pvt. Ltd.'s", "Tata Power Solar Systems's", "TataPSS", "TataPSS's", "TataPowerSolarSystems", "TataPowerSolarSystemsLim", "TataPowerSolarSystemsLim's", "TataPowerSolarSystemsLimited", "TataPowerSolarSystemsLimited's", "tatapowersolarsystems", "tatapowersolarsystemslimited", "tatapowersolarsystemslimited's"       ]
    },

    "ReNew Energy Global PLC": {
        "sbu": "Renewables",
        "aliases": [
        "REGP", "REGP's", "ReNew Energy Global", "ReNew Energy Global Co.", "ReNew Energy Global Co.'s", "ReNew Energy Global Company", "ReNew Energy Global Company's", "ReNew Energy Global LLC", "ReNew Energy Global LLC's", "ReNew Energy Global Limited", "ReNew Energy Global Limited's", "ReNew Energy Global Ltd", "ReNew Energy Global Ltd.", "ReNew Energy Global Ltd.'s", "ReNew Energy Global PLC", "ReNew Energy Global PLC's", "ReNew Energy Global Private Limited", "ReNew Energy Global Private Limited's", "ReNew Energy Global Pvt Ltd", "ReNew Energy Global Pvt Ltd's", "ReNew Energy Global Pvt. Ltd.", "ReNew Energy Global Pvt. Ltd.'s", "ReNew Energy Global's", "ReNewEnergyGlobal", "ReNewEnergyGlobal's", "ReNewEnergyGlobalPLC", "ReNewEnergyGlobalPLC's", "renewenergyglobal", "renewenergyglobal's", "renewenergyglobalplc", "renewenergyglobalplc's"]
    },

    "Azure Power Global Limited": {
        "sbu": "Renewables",
        "aliases": [
        "A&P", "A&P's", "A-P", "A.P", "AP", "APGL", "APGL's", "Azure P", "Azure Power Global", "Azure Power Global Co.", "Azure Power Global Company", "Azure Power Global Company's", "Azure Power Global LLC", "Azure Power Global LLC's", "Azure Power Global Lim", "Azure Power Global Limited", "Azure Power Global Ltd", "Azure Power Global Ltd.", "Azure Power Global Ltd.'s", "Azure Power Global PLC", "Azure Power Global PLC's", "Azure Power Global Private Lim", "Azure Power Global Private Lim's", "Azure Power Global Private Limited", "Azure Power Global Private Limited's", "Azure Power Global Pvt Ltd", "Azure Power Global Pvt Ltd's", "Azure Power Global Pvt. Ltd.", "Azure Power Global Pvt. Ltd.'s", "Azure Power Global's", "AzureP", "AzureP's", "AzurePowerGlobal", "AzurePowerGlobal's", "AzurePowerGlobalLim", "AzurePowerGlobalLim's", "AzurePowerGlobalLimited", "AzurePowerGlobalLimited's", "azurepowerglobal", "azurepowerglobal's", "azurepowergloballimited"]
    },

    "Hero Future Energies Limited": {
        "sbu": "Renewables",
        "aliases": [
        "H&F&E", "H&F&E's", "H-F-E", "H.F.E", "HFE", "HFEL", "HFEL's", "Hero FE", "Hero FE's", "Hero Future Energies", "Hero Future Energies Co.", "Hero Future Energies Company", "Hero Future Energies LLC", "Hero Future Energies LLC's", "Hero Future Energies Lim", "Hero Future Energies Lim's", "Hero Future Energies Limited", "Hero Future Energies Limited's", "Hero Future Energies Ltd", "Hero Future Energies Ltd's", "Hero Future Energies Ltd.", "Hero Future Energies Ltd.'s", "Hero Future Energies PLC", "Hero Future Energies PLC's", "Hero Future Energies Private Lim", "Hero Future Energies Private Limited", "Hero Future Energies Pvt Ltd", "Hero Future Energies Pvt. Ltd.", "Hero Future Energies Pvt. Ltd.'s", "Hero Future Energies's", "HeroFE", "HeroFE's", "HeroFutureEnergies", "HeroFutureEnergies's", "HeroFutureEnergiesLim", "HeroFutureEnergiesLim's", "HeroFutureEnergiesLimited", "HeroFutureEnergiesLimited's", "herofutureenergies", "herofutureenergies's", "herofutureenergieslimited", "herofutureenergieslimited's"]
    },

    "Sterling and Wilson Renewable Energy Limited": {
        "sbu": "Renewables",
        "aliases": ["S&A&W", "S-A-W", "S.A.W", "SAW", "SAW's", "SAWREL", "Sterling & Wilson Renewable Energy Lim", "Sterling & Wilson Renewable Energy Lim's", "Sterling & Wilson Renewable Energy Limited", "Sterling & Wilson Renewable Energy Limited's", "Sterling AW", "Sterling AW's", "Sterling Wilson Renewable Energy Lim", "Sterling Wilson Renewable Energy Lim's", "Sterling Wilson Renewable Energy Limited", "Sterling Wilson Renewable Energy Limited's", "Sterling and Wilson Renewable Energy", "Sterling and Wilson Renewable Energy Co.", "Sterling and Wilson Renewable Energy Co.'s", "Sterling and Wilson Renewable Energy Company", "Sterling and Wilson Renewable Energy Company's", "Sterling and Wilson Renewable Energy LLC", "Sterling and Wilson Renewable Energy LLC's", "Sterling and Wilson Renewable Energy Lim", "Sterling and Wilson Renewable Energy Limited", "Sterling and Wilson Renewable Energy Ltd", "Sterling and Wilson Renewable Energy Ltd's", "Sterling and Wilson Renewable Energy Ltd.", "Sterling and Wilson Renewable Energy Ltd.'s", "Sterling and Wilson Renewable Energy PLC", "Sterling and Wilson Renewable Energy Private Lim", "Sterling and Wilson Renewable Energy Private Limited", "Sterling and Wilson Renewable Energy Pvt Ltd", "Sterling and Wilson Renewable Energy Pvt Ltd's", "Sterling and Wilson Renewable Energy Pvt. Ltd.", "Sterling and Wilson Renewable Energy Pvt. Ltd.'s", "Sterling and Wilson Renewable Energy's", "SterlingAW", "SterlingAW's", "SterlingandWilsonRenewableEnergy", "SterlingandWilsonRenewableEnergy's", "SterlingandWilsonRenewableEnergyLim", "SterlingandWilsonRenewableEnergyLimited", "sterlingandwilsonrenewableenergy", "sterlingandwilsonrenewableenergy's", "sterlingandwilsonrenewableenergylimited"    ]
    },

    "Larsen & Toubro Limited": {
        "sbu": "Renewables",
        "aliases": ["L&&&T", "L&&&T's", "L&T", "L-&-T", "L-&-T's", "L.&.T", "L.&.T's", "LTL", "LTL's", "Larsen & Toubro", "Larsen & Toubro Co.", "Larsen & Toubro Co.'s", "Larsen & Toubro Company", "Larsen & Toubro Company's", "Larsen & Toubro LLC", "Larsen & Toubro Lim", "Larsen & Toubro Lim's", "Larsen & Toubro Limited", "Larsen & Toubro Limited's", "Larsen & Toubro Ltd", "Larsen & Toubro Ltd.", "Larsen & Toubro Ltd.'s", "Larsen & Toubro PLC", "Larsen & Toubro PLC's", "Larsen & Toubro Private Lim", "Larsen & Toubro Private Limited", "Larsen & Toubro Pvt Ltd", "Larsen & Toubro Pvt. Ltd.", "Larsen &T", "Larsen &T's", "Larsen Toubro Lim", "Larsen Toubro Lim's", "Larsen Toubro Limited", "Larsen Toubro Limited's", "Larsen and Toubro Lim", "Larsen and Toubro Limited", "Larsen&T", "Larsen&T's", "Larsen&Toubro", "Larsen&Toubro's", "Larsen&ToubroLim", "Larsen&ToubroLim's", "Larsen&ToubroLimited", "Larsen&ToubroLimited's", "larsen&toubro", "larsen&toubro's", "larsen&toubrolimited"]},
    "Kalpataru Projects International Limited": {
        "sbu": "Renewables",
        "aliases": ["K&P", "K&P's", "K-P", "K-P's", "K.P", "K.P's", "KP", "KPIL", "KPIL's", "Kalpataru P", "Kalpataru P's", "Kalpataru Projects International", "Kalpataru Projects International Co.", "Kalpataru Projects International Company", "Kalpataru Projects International Company's", "Kalpataru Projects International LLC", "Kalpataru Projects International Lim", "Kalpataru Projects International Lim's", "Kalpataru Projects International Limited", "Kalpataru Projects International Limited's", "Kalpataru Projects International Ltd", "Kalpataru Projects International Ltd.", "Kalpataru Projects International Ltd.'s", "Kalpataru Projects International PLC", "Kalpataru Projects International PLC's", "Kalpataru Projects International Private Lim", "Kalpataru Projects International Private Lim's", "Kalpataru Projects International Private Limited", "Kalpataru Projects International Private Limited's", "Kalpataru Projects International Pvt Ltd", "Kalpataru Projects International Pvt Ltd's", "Kalpataru Projects International Pvt. Ltd.", "Kalpataru Projects International's", "KalpataruP", "KalpataruP's", "KalpataruProjectsInternational", "KalpataruProjectsInternational's", "KalpataruProjectsInternationalLim", "KalpataruProjectsInternationalLimited", "kalpataruprojectsinternational", "kalpataruprojectsinternationallimited"]},
    "Bharat Heavy Electricals Limited": {
        "sbu": "Renewables",
        "aliases": ["B&H&E", "B-H-E", "B.H.E", "B.H.E's", "BHE", "BHE's", "BHEL", "BHEL's", "Bharat HE", "Bharat HE's", "Bharat Heavy Electricals", "Bharat Heavy Electricals Co.", "Bharat Heavy Electricals Co.'s", "Bharat Heavy Electricals Company", "Bharat Heavy Electricals Company's", "Bharat Heavy Electricals LLC", "Bharat Heavy Electricals Lim", "Bharat Heavy Electricals Lim's", "Bharat Heavy Electricals Limited", "Bharat Heavy Electricals Limited's", "Bharat Heavy Electricals Ltd", "Bharat Heavy Electricals Ltd.", "Bharat Heavy Electricals PLC", "Bharat Heavy Electricals PLC's", "Bharat Heavy Electricals Private Lim", "Bharat Heavy Electricals Private Limited", "Bharat Heavy Electricals Pvt Ltd", "Bharat Heavy Electricals Pvt Ltd's", "Bharat Heavy Electricals Pvt. Ltd.", "Bharat Heavy Electricals's", "BharatHE", "BharatHE's", "BharatHeavyElectricals", "BharatHeavyElectricals's", "BharatHeavyElectricalsLim", "BharatHeavyElectricalsLim's", "BharatHeavyElectricalsLimited", "BharatHeavyElectricalsLimited's", "bharatheavyelectricals", "bharatheavyelectricals's", "bharatheavyelectricalslimited", "bharatheavyelectricalslimited's"]},
    "Ever Renew Energy Pvt. Ltd.": {
        "sbu": "Renewables",
        "aliases": [
        "E&R", "E-R", "E-R's", "E.R", "E.R's", "ER", "EREPL", "EREPL's", "Ever R", "Ever Renew Energy Pvt.", "Ever Renew Energy Pvt. Co.", "Ever Renew Energy Pvt. Co.'s", "Ever Renew Energy Pvt. Company", "Ever Renew Energy Pvt. Company's", "Ever Renew Energy Pvt. LLC", "Ever Renew Energy Pvt. Limited", "Ever Renew Energy Pvt. Ltd", "Ever Renew Energy Pvt. Ltd's", "Ever Renew Energy Pvt. Ltd.", "Ever Renew Energy Pvt. Ltd.'s", "Ever Renew Energy Pvt. PLC", "Ever Renew Energy Pvt. Private Limited", "Ever Renew Energy Pvt. Private Limited's", "Ever Renew Energy Pvt. Pvt Ltd", "Ever Renew Energy Pvt. Pvt Ltd's", "Ever Renew Energy Pvt. Pvt. Ltd.", "Ever Renew Energy Pvt. Pvt. Ltd.'s", "Ever Renew Energy Pvt.'s", "EverR", "EverR's", "EverRenewEnergyPvt.", "EverRenewEnergyPvt.Ltd.", "EverRenewEnergyPvt.Ltd.'s", "everrenewenergypvt.", "everrenewenergypvt.'s", "everrenewenergypvt.ltd.", "everrenewenergypvt.ltd.'s"]
    },

    "Rays Power Infra India Limited": {
        "sbu": "Renewables",
        "aliases": [
        "R&P", "R&P's", "R-P", "R-P's", "R.P", "RP", "RPIIL", "RPIIL's", "Rays P", "Rays P's", "Rays Power Infra India", "Rays Power Infra India Co.", "Rays Power Infra India Co.'s", "Rays Power Infra India Company", "Rays Power Infra India Company's", "Rays Power Infra India LLC", "Rays Power Infra India Lim", "Rays Power Infra India Lim's", "Rays Power Infra India Limited", "Rays Power Infra India Limited's", "Rays Power Infra India Ltd", "Rays Power Infra India Ltd's", "Rays Power Infra India Ltd.", "Rays Power Infra India PLC", "Rays Power Infra India PLC's", "Rays Power Infra India Private Lim", "Rays Power Infra India Private Lim's", "Rays Power Infra India Private Limited", "Rays Power Infra India Private Limited's", "Rays Power Infra India Pvt Ltd", "Rays Power Infra India Pvt Ltd's", "Rays Power Infra India Pvt. Ltd.", "Rays Power Infra India Pvt. Ltd.'s", "Rays Power Infra India's", "RaysP", "RaysPowerInfraIndia", "RaysPowerInfraIndia's", "RaysPowerInfraIndiaLim", "RaysPowerInfraIndiaLim's", "RaysPowerInfraIndiaLimited", "RaysPowerInfraIndiaLimited's", "rayspowerinfraindia", "rayspowerinfraindialimited"]
    },

    "Jackson Electricals & Infrastructure Pvt. Ltd.": {
        "sbu": "Renewables",
        "aliases": ["J&E&&", "J-E-&", "J-E-&'s", "J.E.&", "JE&", "JE&'s", "JEIPL", "JEIPL's", "Jackson E&", "Jackson E&'s", "Jackson Electricals & Infrastructure Pvt.", "Jackson Electricals & Infrastructure Pvt. Co.", "Jackson Electricals & Infrastructure Pvt. Co.'s", "Jackson Electricals & Infrastructure Pvt. Company", "Jackson Electricals & Infrastructure Pvt. Company's", "Jackson Electricals & Infrastructure Pvt. LLC", "Jackson Electricals & Infrastructure Pvt. LLC's", "Jackson Electricals & Infrastructure Pvt. Limited", "Jackson Electricals & Infrastructure Pvt. Limited's", "Jackson Electricals & Infrastructure Pvt. Ltd", "Jackson Electricals & Infrastructure Pvt. Ltd's", "Jackson Electricals & Infrastructure Pvt. Ltd.", "Jackson Electricals & Infrastructure Pvt. PLC", "Jackson Electricals & Infrastructure Pvt. PLC's", "Jackson Electricals & Infrastructure Pvt. Private Limited", "Jackson Electricals & Infrastructure Pvt. Pvt Ltd", "Jackson Electricals & Infrastructure Pvt. Pvt. Ltd.", "Jackson Electricals & Infrastructure Pvt. Pvt. Ltd.'s", "Jackson Electricals Infrastructure Pvt. Ltd.", "Jackson Electricals Infrastructure Pvt. Ltd.'s", "Jackson Electricals and Infrastructure Pvt. Ltd.", "JacksonE&", "JacksonE&'s", "JacksonElectricals&InfrastructurePvt.", "JacksonElectricals&InfrastructurePvt.'s", "JacksonElectricals&InfrastructurePvt.Ltd.", "jacksonelectricals&infrastructurepvt.", "jacksonelectricals&infrastructurepvt.ltd.", "jacksonelectricals&infrastructurepvt.ltd.'s"
        ]
    },

    "Siemens Energy India Limited": {
        "sbu": "Renewables",
        "aliases": ["SEIL", "SEIL's", "Siemens Energy India", "Siemens Energy India Co.", "Siemens Energy India Co.'s", "Siemens Energy India Company", "Siemens Energy India Company's", "Siemens Energy India LLC", "Siemens Energy India Lim", "Siemens Energy India Lim's", "Siemens Energy India Limited", "Siemens Energy India Limited's", "Siemens Energy India Ltd", "Siemens Energy India Ltd's", "Siemens Energy India Ltd.", "Siemens Energy India Ltd.'s", "Siemens Energy India PLC", "Siemens Energy India PLC's", "Siemens Energy India Private Lim", "Siemens Energy India Private Lim's", "Siemens Energy India Private Limited", "Siemens Energy India Private Limited's", "Siemens Energy India Pvt Ltd", "Siemens Energy India Pvt Ltd's", "Siemens Energy India Pvt. Ltd.", "Siemens Energy India Pvt. Ltd.'s", "Siemens Energy India's", "SiemensEnergyIndia", "SiemensEnergyIndia's", "SiemensEnergyIndiaLim", "SiemensEnergyIndiaLim's", "SiemensEnergyIndiaLimited", "SiemensEnergyIndiaLimited's", "siemensenergyindia", "siemensenergyindia's", "siemensenergyindialimited", "siemensenergyindialimited's"]},
    "Tata Projects Limited": {
        "sbu": "Renewables",
        "aliases": ["T&P", "T&P's", "T-P", "T-P's", "T.P", "T.P's", "TP", "TPL", "TPL's", "Tata P", "Tata Projects", "Tata Projects Co.", "Tata Projects Company", "Tata Projects Company's", "Tata Projects LLC", "Tata Projects Lim", "Tata Projects Lim's", "Tata Projects Limited", "Tata Projects Limited's", "Tata Projects Ltd", "Tata Projects Ltd.", "Tata Projects Ltd.'s", "Tata Projects PLC", "Tata Projects PLC's", "Tata Projects Private Lim", "Tata Projects Private Lim's", "Tata Projects Private Limited", "Tata Projects Private Limited's", "Tata Projects Pvt Ltd", "Tata Projects Pvt Ltd's", "Tata Projects Pvt. Ltd.", "Tata Projects Pvt. Ltd.'s", "Tata Projects's", "TataP", "TataP's", "TataProjects", "TataProjectsLim", "TataProjectsLimited", "tataprojects", "tataprojects's", "tataprojectslimited", "tataprojectslimited's"]},
    "NCC Limited": {
        "sbu": "Renewables",
        "aliases": ["NCC", "NCC Co.", "NCC Co.'s", "NCC Company", "NCC Company's", "NCC LLC", "NCC LLC's", "NCC Lim", "NCC Lim's", "NCC Limited", "NCC Limited's", "NCC Ltd", "NCC Ltd's", "NCC Ltd.", "NCC Ltd.'s", "NCC Private Lim", "NCC Private Lim's", "NCC Private Limited", "NCC Private Limited's", "NCC Pvt Ltd", "NCC Pvt Ltd's", "NCC Pvt. Ltd.", "NCC Pvt. Ltd.'s", "NCC's", "National Construction Company", "National Construction Company Co.", "National Construction Company Co.'s", "National Construction Company Company", "National Construction Company Company's", "National Construction Company LLC", "National Construction Company LLC's", "National Construction Company Lim", "National Construction Company Lim's", "National Construction Company Limited", "National Construction Company Limited's", "National Construction Company Ltd", "National Construction Company Ltd's", "National Construction Company Ltd.", "National Construction Company Ltd.'s", "National Construction Company Private Lim", "National Construction Company Private Lim's", "National Construction Company Private Limited", "National Construction Company Private Limited's", "National Construction Company Pvt Ltd", "National Construction Company Pvt Ltd's", "National Construction Company Pvt. Ltd.", "National Construction Company Pvt. Ltd.'s", "NationalConstructionCompany", "nationalconstructioncompany", "ncc", "ncc's"]},
    "Shapoorji Pallonji & Company Private Limited": {
        "sbu": "Renewables",
        "aliases": ["S&P&&", "S&P&&'s", "S-P-&", "S.P.&", "SP&", "SPCPL", "SPCPL's", "Shapoorji P&", "Shapoorji Pallonji &  Private", "Shapoorji Pallonji &  Private Co.", "Shapoorji Pallonji &  Private Co.'s", "Shapoorji Pallonji &  Private Company", "Shapoorji Pallonji &  Private Company's", "Shapoorji Pallonji &  Private LLC", "Shapoorji Pallonji &  Private LLC's", "Shapoorji Pallonji &  Private Lim", "Shapoorji Pallonji &  Private Limited", "Shapoorji Pallonji &  Private Ltd", "Shapoorji Pallonji &  Private Ltd's", "Shapoorji Pallonji &  Private Ltd.", "Shapoorji Pallonji &  Private Ltd.'s", "Shapoorji Pallonji &  Private PLC", "Shapoorji Pallonji &  Private Private Lim", "Shapoorji Pallonji &  Private Private Limited", "Shapoorji Pallonji &  Private Pvt Ltd", "Shapoorji Pallonji &  Private Pvt Ltd's", "Shapoorji Pallonji &  Private Pvt. Ltd.", "Shapoorji Pallonji &  Private's", "Shapoorji Pallonji & Company Private Lim", "Shapoorji Pallonji & Company Private Lim's", "Shapoorji Pallonji & Company Private Limited", "Shapoorji Pallonji & Company Private Limited's", "Shapoorji Pallonji Company Private Lim", "Shapoorji Pallonji Company Private Lim's", "Shapoorji Pallonji Company Private Limited", "Shapoorji Pallonji Company Private Limited's", "Shapoorji Pallonji and Company Private Lim", "Shapoorji Pallonji and Company Private Lim's", "Shapoorji Pallonji and Company Private Limited", "Shapoorji Pallonji and Company Private Limited's", "ShapoorjiP&", "ShapoorjiP&'s", "ShapoorjiPallonji&CompanyPrivateLim", "ShapoorjiPallonji&CompanyPrivateLimited", "ShapoorjiPallonji&Private", "shapoorjipallonji&companyprivatelimited", "shapoorjipallonji&companyprivatelimited's", "shapoorjipallonji&private", "shapoorjipallonji&private's"]},
    "Likhitha Infrastructure Limited": {
        "sbu": "Renewables",
        "aliases": ["LIL", "LIL's", "Likhitha Infrastructure", "Likhitha Infrastructure Co.", "Likhitha Infrastructure Co.'s", "Likhitha Infrastructure Company", "Likhitha Infrastructure Company's", "Likhitha Infrastructure LLC", "Likhitha Infrastructure Lim", "Likhitha Infrastructure Lim's", "Likhitha Infrastructure Limited", "Likhitha Infrastructure Limited's", "Likhitha Infrastructure Ltd", "Likhitha Infrastructure Ltd's", "Likhitha Infrastructure Ltd.", "Likhitha Infrastructure Ltd.'s", "Likhitha Infrastructure PLC", "Likhitha Infrastructure PLC's", "Likhitha Infrastructure Private Lim", "Likhitha Infrastructure Private Lim's", "Likhitha Infrastructure Private Limited", "Likhitha Infrastructure Private Limited's", "Likhitha Infrastructure Pvt Ltd", "Likhitha Infrastructure Pvt Ltd's", "Likhitha Infrastructure Pvt. Ltd.", "Likhitha Infrastructure Pvt. Ltd.'s", "LikhithaInfrastructure", "LikhithaInfrastructure's", "LikhithaInfrastructureLim", "LikhithaInfrastructureLim's", "LikhithaInfrastructureLimited", "LikhithaInfrastructureLimited's", "likhithainfrastructure", "likhithainfrastructure's", "likhithainfrastructurelimited", "likhithainfrastructurelimited's" ]
    },

    # --- Oil & Gas ---
    "Larsen & Toubro Limited": {
        "sbu": "Oil & Gas",
        "aliases": ["L&&&T", "L&&&T's", "L&T", "L-&-T", "L-&-T's", "L.&.T", "L.&.T's", "LTL", "LTL's", "Larsen & Toubro", "Larsen & Toubro Co.", "Larsen & Toubro Co.'s", "Larsen & Toubro Company", "Larsen & Toubro Company's", "Larsen & Toubro LLC", "Larsen & Toubro Lim", "Larsen & Toubro Lim's", "Larsen & Toubro Limited", "Larsen & Toubro Limited's", "Larsen & Toubro Ltd", "Larsen & Toubro Ltd.", "Larsen & Toubro Ltd.'s", "Larsen & Toubro PLC", "Larsen & Toubro PLC's", "Larsen & Toubro Private Lim", "Larsen & Toubro Private Limited", "Larsen & Toubro Pvt Ltd", "Larsen & Toubro Pvt. Ltd.", "Larsen &T", "Larsen &T's", "Larsen Toubro Lim", "Larsen Toubro Lim's", "Larsen Toubro Limited", "Larsen Toubro Limited's", "Larsen and Toubro Lim", "Larsen and Toubro Limited", "Larsen&T", "Larsen&T's", "Larsen&Toubro", "Larsen&Toubro's", "Larsen&ToubroLim", "Larsen&ToubroLim's", "Larsen&ToubroLimited", "Larsen&ToubroLimited's", "larsen&toubro", "larsen&toubro's", "larsen&toubrolimited"]},
    "Tata Projects Limited": {
        "sbu": "Oil & Gas",
        "aliases": ["T&P", "T&P's", "T-P", "T-P's", "T.P", "T.P's", "TP", "TPL", "TPL's", "Tata P", "Tata Projects", "Tata Projects Co.", "Tata Projects Company", "Tata Projects Company's", "Tata Projects LLC", "Tata Projects Lim", "Tata Projects Lim's", "Tata Projects Limited", "Tata Projects Limited's", "Tata Projects Ltd", "Tata Projects Ltd.", "Tata Projects Ltd.'s", "Tata Projects PLC", "Tata Projects PLC's", "Tata Projects Private Lim", "Tata Projects Private Lim's", "Tata Projects Private Limited", "Tata Projects Private Limited's", "Tata Projects Pvt Ltd", "Tata Projects Pvt Ltd's", "Tata Projects Pvt. Ltd.", "Tata Projects Pvt. Ltd.'s", "Tata Projects's", "TataP", "TataP's", "TataProjects", "TataProjectsLim", "TataProjectsLimited", "tataprojects", "tataprojects's", "tataprojectslimited", "tataprojectslimited's"]},
    "Bharat Heavy Electricals Limited": {
        "sbu": "Oil & Gas",
        "aliases": ["B&H&E", "B-H-E", "B.H.E", "B.H.E's", "BHE", "BHE's", "BHEL", "BHEL's", "Bharat HE", "Bharat HE's", "Bharat Heavy Electricals", "Bharat Heavy Electricals Co.", "Bharat Heavy Electricals Co.'s", "Bharat Heavy Electricals Company", "Bharat Heavy Electricals Company's", "Bharat Heavy Electricals LLC", "Bharat Heavy Electricals Lim", "Bharat Heavy Electricals Lim's", "Bharat Heavy Electricals Limited", "Bharat Heavy Electricals Limited's", "Bharat Heavy Electricals Ltd", "Bharat Heavy Electricals Ltd.", "Bharat Heavy Electricals PLC", "Bharat Heavy Electricals PLC's", "Bharat Heavy Electricals Private Lim", "Bharat Heavy Electricals Private Limited", "Bharat Heavy Electricals Pvt Ltd", "Bharat Heavy Electricals Pvt Ltd's", "Bharat Heavy Electricals Pvt. Ltd.", "Bharat Heavy Electricals's", "BharatHE", "BharatHE's", "BharatHeavyElectricals", "BharatHeavyElectricals's", "BharatHeavyElectricalsLim", "BharatHeavyElectricalsLim's", "BharatHeavyElectricalsLimited", "BharatHeavyElectricalsLimited's", "bharatheavyelectricals", "bharatheavyelectricals's", "bharatheavyelectricalslimited", "bharatheavyelectricalslimited's"]},
    "Hindustan Construction Company Limited": {
        "sbu": "Oil & Gas",
        "aliases": ["H&C", "H&C's", "H-C", "H-C's", "H.C", "H.C's", "HC", "HCCL", "HCCL's", "Hindustan C", "Hindustan C's", "Hindustan Construction", "Hindustan Construction Co.", "Hindustan Construction Company", "Hindustan Construction Company Lim", "Hindustan Construction Company Lim's", "Hindustan Construction Company Limited", "Hindustan Construction Company Limited's", "Hindustan Construction LLC", "Hindustan Construction Lim", "Hindustan Construction Lim's", "Hindustan Construction Limited", "Hindustan Construction Limited's", "Hindustan Construction Ltd", "Hindustan Construction Ltd.", "Hindustan Construction Ltd.'s", "Hindustan Construction PLC", "Hindustan Construction Private Lim", "Hindustan Construction Private Lim's", "Hindustan Construction Private Limited", "Hindustan Construction Private Limited's", "Hindustan Construction Pvt Ltd", "Hindustan Construction Pvt. Ltd.", "Hindustan Construction Pvt. Ltd.'s", "HindustanC", "HindustanC's", "HindustanConstruction", "HindustanConstruction's", "HindustanConstructionCompanyLim", "HindustanConstructionCompanyLim's", "HindustanConstructionCompanyLimited", "HindustanConstructionCompanyLimited's", "hindustanconstruction", "hindustanconstruction's", "hindustanconstructioncompanylimited"]},
    "Ace Pipeline Contracts Private Limited": {
        "sbu": "Oil & Gas",
        "aliases": ["A&P&C", "A&P&C's", "A-P-C", "A.P.C", "APC", "APCPL", "APCPL's", "Ace PC", "Ace PC's", "Ace Pipeline Contracts Private", "Ace Pipeline Contracts Private Co.", "Ace Pipeline Contracts Private Co.'s", "Ace Pipeline Contracts Private Company", "Ace Pipeline Contracts Private Company's", "Ace Pipeline Contracts Private LLC", "Ace Pipeline Contracts Private Lim", "Ace Pipeline Contracts Private Lim's", "Ace Pipeline Contracts Private Limited", "Ace Pipeline Contracts Private Limited's", "Ace Pipeline Contracts Private Ltd", "Ace Pipeline Contracts Private Ltd's", "Ace Pipeline Contracts Private Ltd.", "Ace Pipeline Contracts Private Ltd.'s", "Ace Pipeline Contracts Private PLC", "Ace Pipeline Contracts Private Private Lim", "Ace Pipeline Contracts Private Private Limited", "Ace Pipeline Contracts Private Pvt Ltd", "Ace Pipeline Contracts Private Pvt Ltd's", "Ace Pipeline Contracts Private Pvt. Ltd.", "Ace Pipeline Contracts Private Pvt. Ltd.'s", "Ace Pipeline Contracts Private's", "AcePC", "AcePC's", "AcePipelineContractsPrivate", "AcePipelineContractsPrivate's", "AcePipelineContractsPrivateLim", "AcePipelineContractsPrivateLimited", "acepipelinecontractsprivate", "acepipelinecontractsprivate's", "acepipelinecontractsprivatelimited", "acepipelinecontractsprivatelimited's"]},
    "Corrtech International Limited": {
        "sbu": "Oil & Gas",
        "aliases": ["CIL", "CIL's", "Corrtech International", "Corrtech International Co.", "Corrtech International Co.'s", "Corrtech International Company", "Corrtech International Company's", "Corrtech International LLC", "Corrtech International LLC's", "Corrtech International Lim", "Corrtech International Limited", "Corrtech International Ltd", "Corrtech International Ltd's", "Corrtech International Ltd.", "Corrtech International Ltd.'s", "Corrtech International PLC", "Corrtech International Private Lim", "Corrtech International Private Lim's", "Corrtech International Private Limited", "Corrtech International Private Limited's", "Corrtech International Pvt Ltd", "Corrtech International Pvt Ltd's", "Corrtech International Pvt. Ltd.", "Corrtech International Pvt. Ltd.'s", "Corrtech International's", "CorrtechInternational", "CorrtechInternational's", "CorrtechInternationalLim", "CorrtechInternationalLim's", "CorrtechInternationalLimited", "CorrtechInternationalLimited's", "corrtechinternational", "corrtechinternational's", "corrtechinternationallimited", "corrtechinternationallimited's"]},
    "NCC Limited": {
        "sbu": "Oil & Gas",
        "aliases": ["NCC", "NCC Co.", "NCC Co.'s", "NCC Company", "NCC Company's", "NCC LLC", "NCC LLC's", "NCC Lim", "NCC Lim's", "NCC Limited", "NCC Limited's", "NCC Ltd", "NCC Ltd's", "NCC Ltd.", "NCC Ltd.'s", "NCC Private Lim", "NCC Private Lim's", "NCC Private Limited", "NCC Private Limited's", "NCC Pvt Ltd", "NCC Pvt Ltd's", "NCC Pvt. Ltd.", "NCC Pvt. Ltd.'s", "NCC's", "National Construction Company", "National Construction Company Co.", "National Construction Company Co.'s", "National Construction Company Company", "National Construction Company Company's", "National Construction Company LLC", "National Construction Company LLC's", "National Construction Company Lim", "National Construction Company Lim's", "National Construction Company Limited", "National Construction Company Limited's", "National Construction Company Ltd", "National Construction Company Ltd's", "National Construction Company Ltd.", "National Construction Company Ltd.'s", "National Construction Company Private Lim", "National Construction Company Private Lim's", "National Construction Company Private Limited", "National Construction Company Private Limited's", "National Construction Company Pvt Ltd", "National Construction Company Pvt Ltd's", "National Construction Company Pvt. Ltd.", "National Construction Company Pvt. Ltd.'s", "NationalConstructionCompany", "nationalconstructioncompany", "ncc", "ncc's"]},
    "Shapoorji Pallonji & Company Private Limited": {
        "sbu": "Oil & Gas",
        "aliases": ["S&P&&", "S&P&&'s", "S-P-&", "S.P.&", "SP&", "SPCPL", "SPCPL's", "Shapoorji P&", "Shapoorji Pallonji &  Private", "Shapoorji Pallonji &  Private Co.", "Shapoorji Pallonji &  Private Co.'s", "Shapoorji Pallonji &  Private Company", "Shapoorji Pallonji &  Private Company's", "Shapoorji Pallonji &  Private LLC", "Shapoorji Pallonji &  Private LLC's", "Shapoorji Pallonji &  Private Lim", "Shapoorji Pallonji &  Private Limited", "Shapoorji Pallonji &  Private Ltd", "Shapoorji Pallonji &  Private Ltd's", "Shapoorji Pallonji &  Private Ltd.", "Shapoorji Pallonji &  Private Ltd.'s", "Shapoorji Pallonji &  Private PLC", "Shapoorji Pallonji &  Private Private Lim", "Shapoorji Pallonji &  Private Private Limited", "Shapoorji Pallonji &  Private Pvt Ltd", "Shapoorji Pallonji &  Private Pvt Ltd's", "Shapoorji Pallonji &  Private Pvt. Ltd.", "Shapoorji Pallonji &  Private's", "Shapoorji Pallonji & Company Private Lim", "Shapoorji Pallonji & Company Private Lim's", "Shapoorji Pallonji & Company Private Limited", "Shapoorji Pallonji & Company Private Limited's", "Shapoorji Pallonji Company Private Lim", "Shapoorji Pallonji Company Private Lim's", "Shapoorji Pallonji Company Private Limited", "Shapoorji Pallonji Company Private Limited's", "Shapoorji Pallonji and Company Private Lim", "Shapoorji Pallonji and Company Private Lim's", "Shapoorji Pallonji and Company Private Limited", "Shapoorji Pallonji and Company Private Limited's", "ShapoorjiP&", "ShapoorjiP&'s", "ShapoorjiPallonji&CompanyPrivateLim", "ShapoorjiPallonji&CompanyPrivateLimited", "ShapoorjiPallonji&Private", "shapoorjipallonji&companyprivatelimited", "shapoorjipallonji&companyprivatelimited's", "shapoorjipallonji&private", "shapoorjipallonji&private's"]},
    "Ahluwalia Contracts (India) Limited": {
        "sbu": "Oil & Gas",
        "aliases": ["A&C&(", "A&C&('s", "A-C-(", "A-C-('s", "A.C.(", "A.C.('s", "AC(", "AC('s", "AC(L", "Ahluwalia C(", "Ahluwalia Contracts (India)", "Ahluwalia Contracts (India) Co.", "Ahluwalia Contracts (India) Company", "Ahluwalia Contracts (India) LLC", "Ahluwalia Contracts (India) Lim", "Ahluwalia Contracts (India) Lim's", "Ahluwalia Contracts (India) Limited", "Ahluwalia Contracts (India) Limited's", "Ahluwalia Contracts (India) Ltd", "Ahluwalia Contracts (India) Ltd's", "Ahluwalia Contracts (India) Ltd.", "Ahluwalia Contracts (India) Ltd.'s", "Ahluwalia Contracts (India) PLC", "Ahluwalia Contracts (India) PLC's", "Ahluwalia Contracts (India) Private Lim", "Ahluwalia Contracts (India) Private Lim's", "Ahluwalia Contracts (India) Private Limited", "Ahluwalia Contracts (India) Private Limited's", "Ahluwalia Contracts (India) Pvt Ltd", "Ahluwalia Contracts (India) Pvt Ltd's", "Ahluwalia Contracts (India) Pvt. Ltd.", "Ahluwalia Contracts (India) Pvt. Ltd.'s", "AhluwaliaC(", "AhluwaliaC('s", "AhluwaliaContracts(India)", "AhluwaliaContracts(India)Lim", "AhluwaliaContracts(India)Lim's", "AhluwaliaContracts(India)Limited", "AhluwaliaContracts(India)Limited's", "ahluwaliacontracts(india)", "ahluwaliacontracts(india)'s", "ahluwaliacontracts(india)limited", "ahluwaliacontracts(india)limited's"]},
    "AFCONS Infrastructure Limited": {
        "sbu": "Oil & Gas",
        "aliases": ["AFCONS Infrastructure", "AFCONS Infrastructure Co.", "AFCONS Infrastructure Co.'s", "AFCONS Infrastructure Company", "AFCONS Infrastructure Company's", "AFCONS Infrastructure LLC", "AFCONS Infrastructure LLC's", "AFCONS Infrastructure Lim", "AFCONS Infrastructure Limited", "AFCONS Infrastructure Ltd", "AFCONS Infrastructure Ltd's", "AFCONS Infrastructure Ltd.", "AFCONS Infrastructure Ltd.'s", "AFCONS Infrastructure PLC", "AFCONS Infrastructure PLC's", "AFCONS Infrastructure Private Lim", "AFCONS Infrastructure Private Lim's", "AFCONS Infrastructure Private Limited", "AFCONS Infrastructure Private Limited's", "AFCONS Infrastructure Pvt Ltd", "AFCONS Infrastructure Pvt Ltd's", "AFCONS Infrastructure Pvt. Ltd.", "AFCONS Infrastructure Pvt. Ltd.'s", "AFCONS Infrastructure's", "AFCONSInfrastructure", "AFCONSInfrastructure's", "AFCONSInfrastructureLim", "AFCONSInfrastructureLimited", "AIL", "AIL's", "afconsinfrastructure", "afconsinfrastructure's", "afconsinfrastructurelimited", "afconsinfrastructurelimited's"]},
    "PNC Infratech Limited": {
        "sbu": "Oil & Gas",
        "aliases": ["P&I", "P-I", "P-I's", "P.I", "P.I's", "PI", "PIL", "PNC I", "PNC I's", "PNC Infratech", "PNC Infratech Co.", "PNC Infratech Co.'s", "PNC Infratech Company", "PNC Infratech Company's", "PNC Infratech LLC", "PNC Infratech LLC's", "PNC Infratech Lim", "PNC Infratech Lim's", "PNC Infratech Limited", "PNC Infratech Limited's", "PNC Infratech Ltd", "PNC Infratech Ltd's", "PNC Infratech Ltd.", "PNC Infratech Ltd.'s", "PNC Infratech PLC", "PNC Infratech Private Lim", "PNC Infratech Private Lim's", "PNC Infratech Private Limited", "PNC Infratech Private Limited's", "PNC Infratech Pvt Ltd", "PNC Infratech Pvt. Ltd.", "PNC Infratech's", "PNCI", "PNCI's", "PNCInfratech", "PNCInfratech's", "PNCInfratechLim", "PNCInfratechLim's", "PNCInfratechLimited", "PNCInfratechLimited's", "pncinfratech", "pncinfratechlimited"]},
    "Dilip Buildcon Limited": {
        "sbu": "Oil & Gas",
        "aliases": ["D&B", "D&B's", "D-B", "D-B's", "D.B", "DB", "DBL", "DBL's", "Dilip B", "Dilip B's", "Dilip Buildcon", "Dilip Buildcon Co.", "Dilip Buildcon Co.'s", "Dilip Buildcon Company", "Dilip Buildcon Company's", "Dilip Buildcon LLC", "Dilip Buildcon Lim", "Dilip Buildcon Limited", "Dilip Buildcon Ltd", "Dilip Buildcon Ltd's", "Dilip Buildcon Ltd.", "Dilip Buildcon Ltd.'s", "Dilip Buildcon PLC", "Dilip Buildcon Private Lim", "Dilip Buildcon Private Lim's", "Dilip Buildcon Private Limited", "Dilip Buildcon Private Limited's", "Dilip Buildcon Pvt Ltd", "Dilip Buildcon Pvt Ltd's", "Dilip Buildcon Pvt. Ltd.", "Dilip Buildcon Pvt. Ltd.'s", "Dilip Buildcon's", "DilipB", "DilipBuildcon", "DilipBuildcon's", "DilipBuildconLim", "DilipBuildconLimited", "dilipbuildcon", "dilipbuildconlimited", "dilipbuildconlimited's"]},
    "Siemens Energy India Limited": {
        "sbu": "Oil & Gas",
        "aliases": ["SEIL", "SEIL's", "Siemens Energy India", "Siemens Energy India Co.", "Siemens Energy India Co.'s", "Siemens Energy India Company", "Siemens Energy India Company's", "Siemens Energy India LLC", "Siemens Energy India Lim", "Siemens Energy India Lim's", "Siemens Energy India Limited", "Siemens Energy India Limited's", "Siemens Energy India Ltd", "Siemens Energy India Ltd's", "Siemens Energy India Ltd.", "Siemens Energy India Ltd.'s", "Siemens Energy India PLC", "Siemens Energy India PLC's", "Siemens Energy India Private Lim", "Siemens Energy India Private Lim's", "Siemens Energy India Private Limited", "Siemens Energy India Private Limited's", "Siemens Energy India Pvt Ltd", "Siemens Energy India Pvt Ltd's", "Siemens Energy India Pvt. Ltd.", "Siemens Energy India Pvt. Ltd.'s", "Siemens Energy India's", "SiemensEnergyIndia", "SiemensEnergyIndia's", "SiemensEnergyIndiaLim", "SiemensEnergyIndiaLim's", "SiemensEnergyIndiaLimited", "SiemensEnergyIndiaLimited's", "siemensenergyindia", "siemensenergyindia's", "siemensenergyindialimited", "siemensenergyindialimited's"]},
            

    "Hyundai Engineering & Construction Co.": {
        "sbu": "Oil & Gas",
        "aliases": ["H&E&&&C", "H&E&&&C's", "H-E-&-C", "H.E.&.C", "HE&C", "HECC", "HECC's", "Hyundai E&C", "Hyundai E&C's", "Hyundai Engineering & Construction", "Hyundai Engineering & Construction Co.", "Hyundai Engineering & Construction Co.'s", "Hyundai Engineering & Construction Company", "Hyundai Engineering & Construction Company's", "Hyundai Engineering & Construction LLC", "Hyundai Engineering & Construction Limited", "Hyundai Engineering & Construction Limited's", "Hyundai Engineering & Construction Ltd", "Hyundai Engineering & Construction Ltd's", "Hyundai Engineering & Construction Ltd.", "Hyundai Engineering & Construction Ltd.'s", "Hyundai Engineering & Construction PLC", "Hyundai Engineering & Construction PLC's", "Hyundai Engineering & Construction Private Limited", "Hyundai Engineering & Construction Pvt Ltd", "Hyundai Engineering & Construction Pvt Ltd's", "Hyundai Engineering & Construction Pvt. Ltd.", "Hyundai Engineering & Construction's", "Hyundai Engineering Construction Co.", "Hyundai Engineering Construction Co.'s", "Hyundai Engineering and Construction Co.", "Hyundai Engineering and Construction Co.'s", "HyundaiE&C", "HyundaiE&C's", "HyundaiEngineering&Construction", "HyundaiEngineering&ConstructionCo.", "hyundaiengineering&construction", "hyundaiengineering&constructionco.", "hyundaiengineering&constructionco.'s"]},
    "Hitachi Energy India Limited": {
        "sbu": "Oil & Gas",
        "aliases": ["HEIL", "HEIL's", "Hitachi Energy India", "Hitachi Energy India Co.", "Hitachi Energy India Co.'s", "Hitachi Energy India Company", "Hitachi Energy India Company's", "Hitachi Energy India LLC", "Hitachi Energy India LLC's", "Hitachi Energy India Lim", "Hitachi Energy India Lim's", "Hitachi Energy India Limited", "Hitachi Energy India Limited's", "Hitachi Energy India Ltd", "Hitachi Energy India Ltd's", "Hitachi Energy India Ltd.", "Hitachi Energy India Ltd.'s", "Hitachi Energy India PLC", "Hitachi Energy India PLC's", "Hitachi Energy India Private Lim", "Hitachi Energy India Private Lim's", "Hitachi Energy India Private Limited", "Hitachi Energy India Private Limited's", "Hitachi Energy India Pvt Ltd", "Hitachi Energy India Pvt Ltd's", "Hitachi Energy India Pvt. Ltd.", "Hitachi Energy India Pvt. Ltd.'s", "HitachiEnergyIndia", "HitachiEnergyIndia's", "HitachiEnergyIndiaLim", "HitachiEnergyIndiaLim's", "HitachiEnergyIndiaLimited", "HitachiEnergyIndiaLimited's", "hitachienergyindia", "hitachienergyindia's", "hitachienergyindialimited"]},

# The following competitors were already in the original list but their SBU changed/aliases were updated to include other SBUs.
    
    # Multi-SBU updates for existing companies
    "Larsen & Toubro Limited": {
        "sbu": "Multiple",  # Changed to reflect multiple SBUs
        "aliases": [
            "Larsen & Toubro", "L&T", "L&T Limited", "Larsen and Toubro",
            "L and T", "LnT", "Larsen Toubro", "L&T Power", "L&T Construction",
            "L&T Power Transmission", "L&T Electrical & Automation", "L&T Energy",
            "L&T Infrastructure", "L&T Technology Services", "L&T ECC",
            "L&T Hydrocarbon Engineering", "L&T Heavy Engineering",
            "L&T Power Transmission & Distribution", "L&T PT&D",
            "L&T Infrastructure Projects", "L&T Valves", "L&T Energy Hydrocarbon",
            "L&T Metro Rail", "L&T Realty", "LTIMindtree", "L&T Technology",
            "L&T Defense", "L&T Manufacturing", "L&T Precision Engineering",
            "L&T Shipbuilding", "L&T Finance", "larsen & toubro", "l and t",
            "larsen and toubro", "l&t construction", "l&t power", "l&t pt&d",
            "L&T PT&D International", "L&T International", "L&T HVDC",
            "L&T Middle East", "L&T Global Infrastructure", "L&T Overseas",
            "L&T Energy International", "L&T International Operations",
            "L&T Civil", "L&T Buildings", "L&T Metro Rail Hyderabad",
            "L&T Projects", "L&T Civil Engineering", "L&T Railways",
            "L&T Metro", "L&T Transportation", "L&T Rail", "L&T Transit",
            "L&T Renewables", "L&T Renewable Business", "L&T Solar",
            "L&T Wind", "L&T Battery Storage", "L&T EPC Renewables",
            "L&T Green Energy", "L&T Oil & Gas", "L&T Offshore",
            "L&T Subsea", "L&T Pipeline", "L&T EPC Oil Gas", "L&T Hydrocarbon"
        ]
    },

    "Kalpataru Projects International Limited": {
        "sbu": "Multiple",
        "aliases": [
            "Kalpataru Projects", "Kalpataru International", "KPIL",
            "Kalpataru", "Kalpataru T&D", "Kalpataru Transmission",
            "Kalpataru Projects International", "Kalpataru Power Transmission",
            "Kalpataru EPC", "Kalpataru Building & Factories", "Kalpataru Water",
            "Kalpataru Railways", "Kalpataru Urban Infrastructure",
            "Kalpataru Oil & Gas", "Kalpataru Infrastructure", "KPIL T&D",
            "KPIL Transmission & Distribution", "kalpataru power", "kalpataru projects",
            "Kalpataru Global", "Kalpataru Overseas", "Kalpataru HVDC",
            "Kalpataru International Transmission", "KPIL International",
            "Kalpataru Overseas Projects", "Kalpataru Renewables",
            "Kalpataru Solar", "Kalpataru Renewable Energy"
        ]
    },

    "Tata Projects Limited": {
        "sbu": "Multiple",
        "aliases": [
            "Tata Projects", "Tata Projects Ltd", "TPL", "Tata Construction",
            "Tata Power Transmission", "Tata T&D", "Tata Infrastructure",
            "Tata EPC", "Tata Industrial Infrastructure", "Tata Urban Infrastructure",
            "Tata Utility Services", "Tata Quality Services", "Tata Oil & Gas",
            "Tata Transportation", "Tata Aerospace", "tpl", "tata epc",
            "tata infra", "Tata International", "Tata Overseas",
            "Tata Global Infrastructure", "TPL International",
            "Tata Projects International", "Tata Buildings",
            "Tata Civil Works", "Tata Building & Factories",
            "Tata Commercial Buildings", "Tata Railways", "Tata Metro",
            "Tata Rail", "Tata Metro Rail", "Tata Airport", "Tata Transit",
            "Tata Renewables", "Tata Solar Projects",
            "Tata Renewable Energy", "Tata Hydrocarbon",
            "Tata Offshore", "Tata Subsea", "Tata Oil & Gas Projects"
        ]
    },

    "Sterlite Power Transmission Limited": {
        "sbu": "Multiple",
        "aliases": [
            "Sterlite Power", "Sterlite Power Transmission", "SPL",
            "Sterlite Power Ltd", "Sterlite Global Infrastructure",
            "Sterlite Transmission", "Sterlite Power Grid Ventures", "SPGVL",
            "Resonia", "Sterlite Electric", "Sterlite Power Solutions",
            "Sterlite Power Infrastructure", "Serentica", "IndiGrid",
            "Sterlite Power Conductors", "Sterlite Power EHV Cables",
            "sterlite", "sterlite power", "sterlite grid", "Sterlite Brazil",
            "Sterlite International", "Sterlite Transmission Global",
            "Sterlite Power Overseas"
        ]
    },

    "Bharat Heavy Electricals Limited": {
        "sbu": "Multiple",
        "aliases": [
            "Bharat Heavy Electricals", "BHEL", "BHEL Limited",
            "Bharat Heavy Electrical", "BHEL Transmission", "BHEL Power Systems",
            "BHEL Power", "BHEL T&D", "BHEL Manufacturing", "BHEL Electrical",
            "BHEL Power Sector Northern Region", "BHEL PSNR",
            "BHEL Power Sector Eastern Region", "BHEL PSER",
            "BHEL Power Sector Western Region", "BHEL PSWR",
            "BHEL Power Sector Southern Region", "BHEL PSSR",
            "BHEL Heavy Electrical Plant", "BHEL Renewable Energy",
            "BHEL International", "BHEL Overseas", "BHEL HVDC",
            "BHEL Global", "BHEL International Operations",
            "BHEL Railway", "BHEL Traction", "BHEL Metro",
            "BHEL Transportation", "BHEL Solar", "BHEL Wind",
            "BHEL Green Energy", "BHEL Renewables", "BHEL Oil & Gas",
            "BHEL Offshore", "BHEL Pipeline", "BHEL Energy",
            "BHEL Hydrocarbon"
        ]
    },

    "Siemens Energy India Limited": {
        "sbu": "Multiple",
        "aliases": [
            "Siemens Energy India", "Siemens Energy", "Siemens",
            "Siemens Limited", "Siemens India", "Siemens Power Transmission",
            "Siemens Power Systems", "Siemens T&D", "Siemens India Limited",
            "Siemens Power Electronics", "Siemens Automation", "Siemens Digital",
            "Siemens Smart Infrastructure", "Siemens Industrial Solutions",
            "Siemens Renewables", "Siemens Solar", "Siemens Wind",
            "Siemens Oil & Gas", "Siemens Offshore", "Siemens Energy Systems"
        ]
    },

    "Shapoorji Pallonji & Company Private Limited": {
        "sbu": "Multiple",
        "aliases": [
            "Shapoorji Pallonji", "SP&Co", "Shapoorji Pallonji & Co",
            "S P Pallonji", "Shapoorji Pallonji Group", "SP Construction",
            "Shapoorji Pallonji Infrastructure", "Shapoorji Pallonji Realty",
            "SP Rail", "SP Renewables", "SP Oil & Gas", "SP Energy"
        ]
    },

    "NCC Limited": {
        "sbu": "Multiple",
        "aliases": [
            "NCC", "NCC Ltd", "NCC Limited", "National Construction Company",
            "NCC Infrastructure", "NCC Power", "NCC Electrical Division",
            "NCC T&D", "NCC Transportation", "NCC Railways",
            "NCC Water & Environment", "NCC Irrigation", "NCC Mining",
            "NCC Buildings", "NCC Infra", "NCC Rail", "NCC Metro",
            "NCC Highways", "NCC Bridge", "NCC Tunnel", "NCC Transit",
            "NCC Renewables", "NCC Solar", "NCC Wind", "NCC Oil & Gas",
            "NCC Offshore"
        ]
    },

    "Dilip Buildcon Limited": {
        "sbu": "Multiple",
        "aliases": [
            "Dilip Buildcon", "Dilip Buildcon Ltd", "DBL",
            "Dilip Construction", "Dilip Infrastructure",
            "Dilip Buildcon Limited", "Dilip Projects", "Dilip Rail",
            "Dilip Oil & Gas"
        ]
    },

    "PNC Infratech Limited": {
        "sbu": "Multiple",
        "aliases": [
            "PNC Infratech", "PNC Infratech Ltd", "PNC",
            "PNC Infrastructure", "PNC Construction", "PNC Infra",
            "PNC Projects", "PNC Rail", "PNC Oil & Gas"
        ]
    },

    "Simplex Infrastructures Limited": {
        "sbu": "Multiple",
        "aliases": [
            "Simplex Infrastructures", "Simplex", "Simplex Infrastructure",
            "Simplex Ltd", "Simplex Civil", "Simplex Construction",
            "Simplex Infra", "Simplex Rail", "Simplex Metro",
            "Simplex Transportation"
        ]
    },

    "Ashoka Buildcon Limited": {
        "sbu": "Multiple",
        "aliases": [
            "Ashoka Buildcon", "Ashoka Buildcon Ltd", "ABL",
            "Ashoka Construction", "Ashoka Infrastructure",
            "Ashoka Buildcon Limited", "Ashoka Rail"
        ]
    },

    "HG Infra Engineering Limited": {
        "sbu": "Multiple",
        "aliases": [
            "HG Infra Engineering", "HG Infra", "HGIEL",
            "HG Infrastructure", "HG Civil", "HG Engineering",
            "HG Infra Limited", "HG Rail"
        ]
    },

    "Ahluwalia Contracts (India) Limited": {
        "sbu": "Multiple",
        "aliases": [
            "Ahluwalia Contracts", "Ahluwalia", "ACIL",
            "Ahluwalia Contracts India", "Ahluwalia Construction",
            "Ahluwalia Infrastructure", "Ahluwalia Projects",
            "Ahluwalia Rail", "Ahluwalia Oil & Gas"
        ]
    },

    "AFCONS Infrastructure Limited": {
        "sbu": "Multiple",
        "aliases": [
            "AFCONS", "AFCONS Infrastructure", "AFCONS Ltd",
            "AFCONS Limited", "AFCONS Civil", "AFCONS Engineering",
            "AFCONS Construction", "AFCONS Rail", "AFCONS Metro",
            "AFCONS Oil & Gas", "AFCONS Offshore"
        ]
    },

    "Medha Servo Drives Limited": {
        "sbu": "Multiple",
        "aliases": [
            "Medha Servo Drives", "Medha Servo", "MSDL", "Medha Ltd",
            "Medha Electrical", "Medha Power Drives", "Medha Traction",
            "Medha Transportation"
        ]
    },
    
    # The following companies were updated to reflect multiple SBUs
    "Hindustan Construction Company Limited": {
        "sbu": "Multiple",
        "aliases": [
            "Hindustan Construction", "HCC", "HCC Limited",
            "Hindustan Construction Co", "HCC Infrastructure", "HCC Civil",
            "HCC Buildings", "HCC Water Supply", "HCC Irrigation",
            "HCC Transportation", "HCC Railways", "HCC Mass Rapid Transit",
            "HCC Rail", "HCC Metro", "HCC Oil & Gas", "HCC Offshore",
            "HCC Energy", "HCC Hydrocarbon"
        ]
    },

    "Hitachi Energy India Limited": {
        "sbu": "Multiple",
        "aliases": [
            "Hitachi Energy India", "Hitachi Energy", "Hitachi Power Systems",
            "Hitachi ABB Power Systems India", "HEPSI", "HEIL",
            "Hitachi Energy Transmission", "Hitachi Power Grids",
            "Hitachi ABB Power Grids", "Hitachi HVDC", "Hitachi Power Products",
            "Hitachi Power Solutions", "Hitachi Grid Automation",
            "Hitachi Services Business Unit", "Hitachi Oil & Gas",
            "Hitachi Energy Offshore"
        ]
    },

    # Companies that were already in the original list and now have a new SBU
    "Adani Energy Solutions Limited": {
        "sbu": "Transmission", # Retained original SBU as it wasn't contradicted
        "aliases": [
            "adani", "adani energy", "adani transmission"
        ]
    },

    # Companies not present in the new list but in the original
    # (The original prompt stated "Add all other competitors below...")
    "Transmission & EPC": {
        "sbu": "Transmission & EPC",
        "aliases": [
            "tata projects", "tpl", "tata epc", "tata infra"
        ]
    },
    "Energy Systems": {
        "sbu": "Energy Systems",
        "aliases": [
            "siemens", "siemens energy", "siemens grid"
        ]
    },

}

# --- NEW: Preprocessing step to create reverse-lookup ---
COMPETITOR_ALIAS_MAP = {}
for primary_name, data in COMPETITOR_MASTER.items():
    # Map the primary name to itself
    COMPETITOR_ALIAS_MAP[primary_name.lower()] = primary_name
    # Map all aliases to the primary name
    for alias in data["aliases"]:
        COMPETITOR_ALIAS_MAP[alias.lower()] = primary_name

# This set will be used for quick checking if a word is a competitor or alias
COMPETITORS_SET = set(COMPETITOR_ALIAS_MAP.keys())
# -------------------------

# -----------------------------------
# 3️⃣ Detection Functions (Corrected)
# -----------------------------------
def detect_sbu(title: str) -> str:
    t = title.lower()
    matched_sbus = set()
    
    # Collect all matching SBUs
    for sbu, keywords in SBU_KEYWORD_MAP.items():
        for kw in keywords:
            # Simple check for keywords in title
            if kw.lower() in t:
                matched_sbus.add(sbu)
                break # Move to the next SBU once a keyword is found
    
    if matched_sbus:
        # Return all matched SBUs as a comma-separated string
        return ", ".join(sorted(list(matched_sbus)))
    else:
        return "Unclassified"


def detect_competitor(title: str) -> str:
    # Use the pre-processed map to find matches
    t = title.lower()
    matched_competitors = set()
    
    # Check against all aliases and primary names in the pre-processed set
    for alias_or_name in COMPETITORS_SET:
        if alias_or_name in t:
            # Look up the primary name using the alias map
            primary_name = COMPETITOR_ALIAS_MAP.get(alias_or_name)
            if primary_name:
                matched_competitors.add(primary_name)
    
    if matched_competitors:
        # Return all matched primary competitor names as a comma-separated string
        # This prevents over-simplification (e.g., if both L&T and Tata Projects are mentioned)
        return ", ".join(sorted(list(matched_competitors)))
    else:
        return ""

# -----------------------------------
# 4️⃣ Main Keywords for News Search
# -----------------------------------
# ============================================================
KEYWORDS = [
    "transmission", "distribution", "T&D", "power grid", "substation", "circuit breaker",
    "GIS", "Gas Insulated Switchgear", "disconnect switch", "transformer", "MVA",
    "MW capacity", "transmission line", "distribution line", "circuit km", "overhead line",
    "OHL", "underground cable", "HVDC", "High Voltage DC", "AC transmission",
    "power evacuation", "tower", "pylon", "pole", "conductor", "insulator", "earthing",
    "switchgear", "switchyard", "protection relay", "metering", "SCADA",
    "Advanced Metering Infrastructure", "AMI", "smart meter", "grid digitalization",
    "grid modernization", "grid automation", "power quality", "reactive power",
    "harmonic distortion", "transmission infrastructure", "distribution infrastructure",
    "renewable energy integration", "grid stability", "frequency regulation",
    "India transmission project", "India distribution project", "power transmission order",
    "distribution order", "EPC contract", "tender", "RFQ", "RFP", "bid", "award",
    "execution", "commissioning", "PowerGrid", "PGCIL", "Power Grid Corporation of India",
    "CEA", "Central Electricity Authority", "NTPC", "national thermal power",
    "renewable energy transmission", "National Electricity Plan", "NEP", "capex",
    "investment", "ckm", "circuit kilometer", "transmission capacity",
    "substation capacity", "GVA", "GW", "capacity addition", "transmission expansion",
    "intra-state transmission", "inter-state transmission", "state transmission utility",
    "STTU", "distribution utility", "Northern Region", "Southern Region", "Western Region",
    "Eastern Region", "North Eastern Region", "Delhi", "Mumbai", "Chennai", "Kolkata",
    "Bangalore", "Hyderabad", "HVAC", "high voltage AC", "medium voltage", "low voltage",
    "last-mile connectivity", "grid access", "evacuation", "connectivity", "MEP",
    "Mechanical Electrical Plumbing", "civil works", "DPR", "detailed project report",
    "feasibility study", "hybrid transmission", "solar integration", "wind integration",
    "battery storage", "BESS", "energy storage system", "grid balancing",
    "ancillary services", "transmission losses", "technical specifications",
    "performance standards", "quality assurance",

    # --- International Transmission Keywords ---
    "international order", "overseas contract", "export order", "international project",
    "Middle East", "Saudi Arabia", "UAE", "Qatar", "Kuwait", "Oman", "Bahrain", "GCC",
    "Africa", "Kenya", "Tanzania", "Nigeria", "South Africa", "Uganda", "Ethiopia",
    "Vietnam", "Indonesia", "Thailand", "Philippines", "Malaysia", "Singapore", "Sri Lanka",
    "Bangladesh", "Pakistan", "Uzbekistan", "Kazakhstan", "Turkmenistan",
    "Brazil", "Mexico", "Argentina", "Chile", "Colombia", "Europe", "Turkey", "Poland",
    "HVDC link", "subsea cable", "underground transmission",

    # --- Civil Engineering Keywords ---
    "civil engineering", "highway", "bridge", "tunnel", "airport", "runway", "terminal",
    "port", "building", "high-rise", "residential", "stadium", "water supply", "sewerage",
    "wastewater treatment", "urban development", "metro station", "foundation", "concrete",
    "steel", "geotechnical investigation", "project management", "quality control",
    "safety compliance", "PPP", "BOT", "infrastructure financing",

    # --- Railway Keywords ---
    "railway", "rail", "train", "metro", "monorail", "HSR", "bullet train", "station",
    "depot", "rail track", "signaling", "ATC", "ticketing", "rolling stock", "coach",
    "traction system", "catenary", "pantograph", "OCS", "fire safety", "maintenance",
    "civil works (rail)", "OEM", "India metro", "DMRC", "Mumbai Metro",

    # --- Oil & Gas Keywords ---
    "oil", "gas", "LNG", "upstream", "midstream", "downstream", "refining",
    "petrochemical", "drilling", "pipeline", "offshore", "FPSO", "subsea", "manifold",
    "flowline", "hydrostatic test", "refinery", "petrochemical complex", "FEED",
    "EPCI", "HSE", "Saudi Aramco", "ADNOC", "QatarEnergy", "PDO",

    # --- Renewable (Solar, Wind, Storage) Keywords ---
    "solar", "photovoltaic", "PV", "solar panel", "solar farm", "solar project",
    "MW capacity", "GW", "inverter", "transformer", "HT cable", "tracker", "wind energy",
    "wind turbine", "WTG", "BESS", "battery energy storage system",
    "hybrid system", "SCADA", "LCOE", "NTPC Renewable Energy", "SECI",

    # --- EPC Keywords ---
    "EPC", "engineering procurement construction", "turnkey", "contract award", "bid win",
    "order value", "project delivery", "QA", "QC", "vendor", "supplier",
    "contract agreement", "HSE", "international EPC", "joint venture", "automation",
    "digitalization", "sustainability", "ESG"
]


KEYWORDS = [k.lower() for k in KEYWORDS]



print("Fetching news for:", KEYWORDS, "\n")

all_rows = []

# -----------------------------------
# 5️⃣ Fetch News
# -----------------------------------
for keyword in KEYWORDS:

    encoded_keyword = quote(keyword)

    RSS_URL = (
        f"https://news.google.com/rss/search?q={encoded_keyword}+when:{last_days}d&hl=en-IN&gl=IN&ceid=IN:en"
    )

    print(f"Fetching: {RSS_URL}")

    feed = feedparser.parse(RSS_URL)

    for entry in feed.entries:
        title = entry.get("title", "")
        link = entry.get("link", "")
        pubdate = entry.get("published", "")

        # Convert date
        try:
            pubdate_dt = datetime(*entry.published_parsed[:6])
        except:
            pubdate_dt = None

        if pubdate_dt is None:
            continue

        # Extract source
        source = ""
        if "description" in entry:
            soup = BeautifulSoup(entry.description, "html.parser")
            font_tag = soup.find("font")
            if font_tag:
                source = font_tag.text.strip()

        # Filter by keyword
        t = title.lower()
        s = source.lower()

        if not any(k in t or k in s for k in KEYWORDS):
            continue

        # -------------------------
        # Add SBU + Competitor tags
        # -------------------------
        sbu = detect_sbu(title)
        competitor = detect_competitor(title)

        all_rows.append({
            "keyword": keyword,
            "newstitle": title,
            "source": source,
            "link": link,
            "publishedate": pubdate_dt,
            "SBU": sbu,
            "Competitor": competitor
        })

# -------------------------------
# 6️⃣ Remove Duplicates & Export
# -------------------------------
print("\nFinished fetching. Removing duplicates...")

df = pd.DataFrame(all_rows)
df = df.drop_duplicates(subset=["newstitle", "link"])

df.to_excel(OUTPUT_EXCEL, index=False)

print("Done! Output saved:", OUTPUT_EXCEL)
print("Total unique news:", len(df))
