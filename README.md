import streamlit as st
from groq import Groq
import os
from dotenv import load_dotenv
from init_db import init_database
from db import TribesDB
from difflib import get_close_matches

# Load environment variables
env_path = os.path.join(os.path.dirname(__file__), "..", ".env")
load_dotenv(env_path)

# --- 1. CORE CONFIG ---
try:
    API_KEY = st.secrets.get("GROQ_API_KEY") or os.getenv("GROQ_API_KEY")
except:
    API_KEY = os.getenv("GROQ_API_KEY")

if not API_KEY:
    st.error("❌ GROQ_API_KEY not found! Please set it in .env file")
    st.stop()

client = Groq(api_key=API_KEY)

# Initialize database
db_path = os.path.join(os.path.dirname(__file__), "tribes.db")
if not os.path.exists(db_path):
    init_database()

# --- 2. ELEGANT WHITE & GOLD CSS ---
st.markdown("""
<style>
    @import url('https://fonts.googleapis.com/css2?family=Cormorant+Garamond:wght@300;400;500&family=Montserrat:wght@300;400&display=swap');
    
    :root {
        --gold: #C9A227;
        --gold-light: #E8D48B;
        --gold-pale: #F5EDD3;
        --white: #FFFFFF;
        --white-cream: #FDFBF7;
        --text: #2C2C2C;
        --text-light: #6B6B6B;
    }
    
    .stApp {
        background: linear-gradient(180deg, #FFFFFF 0%, #FDFBF7 50%, #FAF8F3 100%);
        font-family: 'Montserrat', sans-serif;
    }
    
    /* Main Title */
    .main-title {
        text-align: center;
        padding: 2.5rem 1rem 2rem;
        position: relative;
    }
    
    .main-title::before {
        content: "◇ ◆ ◇";
        display: block;
        color: var(--gold);
        font-size: 0.7rem;
        letter-spacing: 15px;
        margin-bottom: 1rem;
    }
    
    .main-title h1 {
        font-family: 'Cormorant Garamond', serif !important;
        font-weight: 300 !important;
        font-size: 2.8rem !important;
        color: #2C2C2C !important;
        letter-spacing: 8px !important;
        text-transform: uppercase;
        margin: 0 !important;
    }
    
    .main-title .subtitle {
        color: var(--gold);
        font-size: 0.7rem;
        letter-spacing: 6px;
        text-transform: uppercase;
        margin-top: 0.8rem;
        font-weight: 400;
    }
    
    /* Decorative Line */
    .deco-line {
        height: 1px;
        background: linear-gradient(90deg, transparent, var(--gold-pale), var(--gold), var(--gold-pale), transparent);
        margin: 1.5rem 3rem;
    }
    
    /* Cards */
    .elegant-card {
        background: #FFFFFF;
        border: 1px solid #F5EDD3;
        padding: 1.5rem;
        margin: 1rem 0;
        position: relative;
    }
    
    .elegant-card::before {
        content: "";
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        height: 3px;
        background: linear-gradient(90deg, var(--gold-pale), var(--gold), var(--gold-pale));
    }
    
    /* Sidebar */
    [data-testid="stSidebar"] {
        background: #FDFBF7;
        border-right: 1px solid #F5EDD3;
    }
    
    [data-testid="stSidebar"] .stRadio > label {
        color: #2C2C2C !important;
    }
    
    /* Buttons */
    .stButton > button {
        background: linear-gradient(135deg, #C9A227 0%, #E8D48B 100%) !important;
        color: #FFFFFF !important;
        border: none !important;
        border-radius: 0 !important;
        padding: 0.7rem 2rem !important;
        font-family: 'Montserrat', sans-serif !important;
        font-weight: 400 !important;
        letter-spacing: 3px !important;
        text-transform: uppercase !important;
        font-size: 0.7rem !important;
        transition: all 0.3s ease !important;
    }
    
    .stButton > button:hover {
        background: linear-gradient(135deg, #B8911F 0%, #C9A227 100%) !important;
        transform: translateY(-2px);
        box-shadow: 0 5px 20px rgba(201, 162, 39, 0.3);
    }
    
    /* Input */
    .stTextInput input {
        background: #FFFFFF !important;
        border: 1px solid #F5EDD3 !important;
        border-radius: 0 !important;
        padding: 0.8rem 1rem !important;
        font-family: 'Montserrat', sans-serif !important;
    }
    
    .stTextInput input:focus {
        border-color: var(--gold) !important;
    }
    
    /* Selectbox */
    .stSelectbox > div > div {
        background: #FFFFFF !important;
        border: 1px solid #F5EDD3 !important;
        border-radius: 0 !important;
    }
    
    /* Headers */
    .elegant-header {
        font-family: 'Cormorant Garamond', serif !important;
        font-weight: 400 !important;
        color: #2C2C2C !important;
        letter-spacing: 3px !important;
        text-transform: uppercase !important;
    }
    
    /* Info boxes */
    .stWarning {
        background: #FFFBF0;
        border-left: 3px solid var(--gold);
    }
    
    .stError {
        background: #FFF5F5;
    }
    
    .stSuccess {
        background: #F0FFF4;
    }
    
    .stInfo {
        background: #FAF8F3;
        border-left: 3px solid var(--gold);
    }
    
    /* Results */
    .results-box {
        background: #FFFFFF;
        border: 1px solid #F5EDD3;
        padding: 2rem;
        margin-top: 1.5rem;
    }
    
    .results-box .stMarkdown h2, 
    .results-box .stMarkdown h3 {
        font-family: 'Cormorant Garamond', serif !important;
        color: var(--gold) !important;
    }
    
    /* Footer decorative */
    .footer-deco {
        text-align: center;
        padding: 2rem;
        color: var(--gold-light);
        font-size: 0.6rem;
        letter-spacing: 8px;
    }
    
    /* Section markers */
    .section-marker {
        color: var(--gold);
        font-size: 0.8rem;
        letter-spacing: 4px;
        margin-bottom: 0.5rem;
    }
</style>
""", unsafe_allow_html=True)

# --- 3. HELPER FUNCTIONS ---
def get_tribe_suggestions(user_input):
    all_tribes = TribesDB.get_all_tribes()
    tribe_names = [tribe['name'] for tribe in all_tribes]
    suggestions = get_close_matches(user_input, tribe_names, n=5, cutoff=0.6)
    return suggestions

# --- 4. UI ---
st.set_page_config(
    page_title="DNA Shezhire AI", 
    page_icon="◇",
    layout="wide"
)

# Main Title
st.markdown("""
<div class="main-title">
    <h1>DNA Shezhire</h1>
    <p class="subtitle">Интеллектуальный анализ происхождения</p>
</div>
""", unsafe_allow_html=True)

st.markdown('<div class="deco-line"></div>', unsafe_allow_html=True)

# Sidebar
st.sidebar.markdown('<p class="section-marker">◇ КАТАЛОГ ◇</p>', unsafe_allow_html=True)

juz_options = ["Все жузы", "Старший Жуз", "Средний Жуз", "Младший Жуз"]
selected_juz = st.sidebar.radio(
    "",
    options=juz_options,
    index=0,
    label_visibility="collapsed"
)

# Get tribes
all_tribes = TribesDB.get_all_tribes()

if selected_juz == "Все жузы":
    filtered_tribes = all_tribes
else:
    juz_mapping = {
        "Старший Жуз": "Старший жуз",
        "Средний Жуз": "Средний жуз",
        "Младший Жуз": "Младший жуз"
    }
    db_juz = juz_mapping[selected_juz]
    filtered_tribes = [tribe for tribe in all_tribes if tribe['juz'] == db_juz]

tribe_options = {f"{tribe['name']} ({tribe['juz']})": tribe for tribe in filtered_tribes}

st.sidebar.markdown("")
selected_from_sidebar = st.sidebar.selectbox(
    "Выберите род:",
    options=list(tribe_options.keys()),
    index=None,
    placeholder="◇ Поиск рода...",
    label_visibility="collapsed"
)

# Main content
st.markdown('<p class="section-marker">◇ ПОИСК ◇</p>', unsafe_allow_html=True)
col1, col2 = st.columns([3, 1])

with col1:
    tribe_input = st.text_input(
        "", 
        placeholder="Введите название рода...",
        label_visibility="collapsed"
    ).strip()

with col2:
    analyze_btn = st.button("Анализ")

# Analyze
tribe_to_analyze = None

if selected_from_sidebar:
    tribe_to_analyze = tribe_options[selected_from_sidebar]
else:
    if tribe_input:
        tribe_to_analyze = TribesDB.get_tribe_by_name(tribe_input)

should_analyze = (selected_from_sidebar is not None) or (analyze_btn and tribe_to_analyze is not None)

if should_analyze:
    if tribe_to_analyze is None and tribe_input:
        suggestions = get_tribe_suggestions(tribe_input)
        
        if suggestions:
            st.warning(f"Род '{tribe_input}' не найден")
            st.info("Похожие роды:")
            
            selected_tribe = st.radio(
                "",
                suggestions,
                label_visibility="collapsed",
                horizontal=True
            )
            
            if st.button("Выбрать"):
                tribe_to_analyze = TribesDB.get_tribe_by_name(selected_tribe)
                if tribe_to_analyze:
                    st.session_state.selected_tribe = tribe_to_analyze
        else:
            results = TribesDB.search_tribe(tribe_input)
            if results:
                st.info(f"Найдено: {len(results)}")
                for result in results:
                    st.write(f"• {result['name']} ({result['juz']})")
                tribe_to_analyze = results[0]
            else:
                st.error(f"Род '{tribe_input}' не найден")
    
    if 'selected_tribe' in st.session_state and tribe_to_analyze is None:
        tribe_to_analyze = st.session_state.selected_tribe
        del st.session_state.selected_tribe
    
    if tribe_to_analyze:
        tribe = tribe_to_analyze
        
        st.markdown('<div class="deco-line"></div>', unsafe_allow_html=True)
        
        # Tribe card
        st.markdown(f"""
        <div class="elegant-card">
            <h2 class="elegant-header" style="font-size: 1.5rem;">◇ {tribe['name']} ◇</h2>
            <p style="color: #6B6B6B; margin-top: 0.5rem;">Жуз: {tribe['juz']} | Гаплогруппа: {tribe['haplogroup'] if tribe['haplogroup'] else '—'}</p>
        </div>
        """, unsafe_allow_html=True)
        
        # Clans
        clans = TribesDB.get_tribe_clans(tribe['id'])
        if clans:
            st.markdown('<p class="section-marker">◇ ДРЕВО РОДОВ ◇</p>', unsafe_allow_html=True)
            
            clan_options = [f"{clan['clan_name']}" for clan in clans]
            selected_clan_text = st.selectbox(
                "Выберите род:",
                options=clan_options,
                key=f"clan_{tribe['id']}"
            )
            
            if selected_clan_text:
                selected_clan = next((c for c in clans if c['clan_name'] == selected_clan_text), None)
                
                if selected_clan:
                    st.markdown(f"""
                    <div class="elegant-card" style="margin-left: 1rem;">
                        <h3 style="font-size: 1.1rem;">◆ {selected_clan['clan_name']}</h3>
                    </div>
                    """, unsafe_allow_html=True)
                    
                    if selected_clan['sub_clans']:
                        st.write("**Подразделения:**")
                        sub_clans_list = [sc.strip() for sc in str(selected_clan['sub_clans']).split(',')]
                        for sc in sub_clans_list:
                            st.write(f"  • {sc}")
                    
                    clan_context = f"\nРод: {selected_clan['clan_name']}"
                    if selected_clan['sub_clans']:
                        clan_context += f"\nПодразделения: {selected_clan['sub_clans']}"
        
        # AI Analysis
        st.markdown("")
        with st.spinner("Анализ..."):
            prompt = f"""
Ты — аналитическая система DNA-Shezhire.

АНАЛИЗИРУЕМЫЙ РОД: {tribe['name']} (Жуз: {tribe['juz']})
Гаплогруппа: {tribe['haplogroup'] if tribe['haplogroup'] else 'неопределенная'}
{clan_context if 'clan_context' in locals() else ''}

Выдай 3 БЛОКА:

🧬 ГЕНЕТИЧЕСКИЙ ПРОФИЛЬ
- Частота гаплогруппы
- Уникальные маркеры

⚔️ ЭПОХА ФОРМИРОВАНИЯ
- Исторические события
- Миграции

🔬 СОВРЕМЕННАЯ СТАТИСТИКА
- Распределение
- Связи
"""
            
            try:
                message = client.chat.completions.create(
                    model="llama-3.3-70b-versatile",
                    messages=[{"role": "user", "content": prompt}],
                    max_tokens=1024
                )
                
                st.markdown('<div class="deco-line"></div>', unsafe_allow_html=True)
                
                st.markdown("""
                <div class="results-box">
                    <p class="section-marker">◇ ИССЛЕДОВАНИЕ ◇</p>
                </div>
                """, unsafe_allow_html=True)
                
                st.markdown(message.choices[0].message.content)
                
            except Exception as e:
                st.error(f"Ошибка: {e}")

# Footer
st.markdown('<div class="deco-line"></div>', unsafe_allow_html=True)
st.markdown("""
<div class="footer-deco">
    ◇ ◆ ◇ DNA SHEZHIRE ◇ ◆ ◇
</div>
""", unsafe_allow_html=True)
