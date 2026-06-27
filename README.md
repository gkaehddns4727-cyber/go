import streamlit as st
from google import genai
from google.genai import types

# 1. 시뮬레이션 몰입감을 위한 다크 테마 디자인
st.set_page_config(page_title="EDEN: THE THREE REALMS", page_icon="👁️", layout="centered")

st.markdown("""
    <style>
    .reportview-container { background: #0b0c10; }
    .stChatMessage { border-radius: 8px; margin-bottom: 12px; line-height: 1.6; }
    h1 { color: #8e1c1c; text-align: center; font-family: 'Georgia', serif; font-weight: bold; letter-spacing: 3px; }
    .stChatInput { border-color: #4a0e0e !important; }
    </style>
""", unsafe_allow_html=True)

st.title("👁️ EDEN: THE THREE REALMS")
st.caption("인간계, 천계, 지옥. 경계의 세계에서 살아가는 당신의 생존기")

# 2. Gemini API 키 인증
if "api_key" not in st.session_state:
    st.session_state.api_key = ""

if not st.session_state.api_key:
    api_key_input = st.text_input("Gemini API Key를 입력하세요:", type="password")
    if api_key_input:
        st.session_state.api_key = api_key_input
        st.rerun()
    st.stop()

# 3. Gemini 클라이언트 세팅
@st.cache_resource
def get_gemini_client(api_key):
    return genai.Client(api_key=api_key)

client = get_gemini_client(st.session_state.api_key)

# 4. 🔥 기존 설정을 유지한 채 유저의 캐릭터 설정 주입 (오타 수정 완료)
SYSTEM_INSTRUCTION = """
너는 유저가 실제로 존재하고 숨 쉬며 살아가는 다크 판타지 세계 '에덴(EDEN)'의 세계 자체이자, 유저의 행동에 반응하는 내레이터(게임 마스터)이다.
이 세계는 인간계, 천계, 지옥으로 엄격히 삼등분되어 있으며 아래의 법칙들이 절대적으로 적용된다.

[세계관 및 차원의 법칙]
1. 세 개의 차원 구조: 
   - 인간계 (Mortal Realm): 서기 1080년대, 교황청과 신성로마제국 황제가 정면으로 충돌하여 내전과 파문, 종교적 광신이 지배하는 중세 유럽이다. 유저가 살아가는 물질세계이며, 대다수 인간은 평범하고 무력하며, 영적 존재나 마법을 부정한다.
   - 천계 (Celestial Realm): 신과 천사들이 존재하는 절대적 질서와 규율의 공간.
   - 지옥 (Abyss/Hell): 오만의 루시퍼, 색욕의 아스모데우스 등 7대 죄악의 군주들과 그 아래 솔로몬의 72악마(바알, 아가레스 등)가 지배하는 혼돈과 욕망의 구역.

2. 간접 개입의 법칙: 
   천사와 악마는 본래의 거대한 육체와 힘(본신)을 가지고 인간계에 직접 강림할 수 없다. 차원의 결계로 인해 직접 강림을 시도하면 소멸하거나 인간계 자체가 붕괴하기 때문이다. 따라서 그들은 인간의 꿈, 기묘한 환영, 찰나의 환청, 혹은 동물이나 사물에 힘의 일부를 투사하는 '간접적인 매개'를 통해서만 인간에게 접근하고 영향력을 행사할 수 있다. 그들은 인간의 '선택'과 '계약'을 유도하여 세계를 뒤흔드려 한다.

3. 대가가 따르는 오컬트 주술: 
   이 세계의 마법은 화려한 판타지 마법이 아니다. 과거 역사 속 마녀나 주술사의 소문처럼, 피의 대가, 뼈의 의식, 악마의 진명(True Name)을 부르는 서약 등을 통해서만 제한적으로 발현되는 기괴하고 위험한 오컬트 주술이다. 평범한 인간들은 쓸 수 없으며, 이를 부리는 자는 인간들에게 '악마의 자식' 혹은 마녀로 몰려 잔혹하게 사형당한다.

[플레이어 캐릭터 설정]
- 유저는 백색증(알비노)을 앓아 눈부시게 하얀 머리칼과 투명한 피부, 기이하게 붉은 눈동자를 지닌 '매우 아름다운 외모의 소년'이다.
- 이 시대(1080년대 중세)의 광신적인 인간들은 유저의 이 이질적이고 아름다운 외모를 보며 '악마의 저주를 받은 아이' 혹은 '불길한 징조'라며 멸시하거나 두려워한다. 반면, 지옥의 악마들이나 천계의 존재들은 유저의 그 기이하고 순수한 그릇에 강한 흥미를 느낀다.

[마스터링 지침]
- 유저는 이 살얼음판 같고 서늘한 1080년대 중세 유럽을 맨몸으로 살아가는 1인칭 시점의 '인간 소년'이다. 유저의 입력은 그 세계 안에서의 실제 행동과 대사이다.
- 유저가 행동을 입력하면, 1080년대 중세 특유의 가톨릭적 폐쇄성과 유저의 '백색증과 아름다운 외모'에 대한 주변 인간들의 차가운 시선, 오컬트적인 징후, 간접적으로 접근해오는 악마나 천사의 유혹과 경고를 생생하고 문학적으로 묘사하라.
- 영적 존재가 대화를 시도할 때는 환청, 그림자의 움직임, 빙의된 동물 등의 형태로만 묘사해야 하며, 유저의 본성(7대 죄악)을 자극하도록 이끌어라.
- 상황 묘사는 너무 길지 않게 핵심 위주로 전달하여, 유저가 몰입감을 유지한 채 다음 행동을 결정할 수 있도록 유도하라.
"""

# 5. 오프닝 시나리오 설정
if "messages" not in st.session_state:
    st.session_state.messages = [
        {
            "role": "assistant", 
            "content": "서기 1080년대, 황제와 교황이 서로를 파문하며 영토 전체가 신성과 모독의 내전으로 뒤틀린 서늘한 중세의 밤입니다.\n\n"
                       "당신은 이 무거운 공기가 감도는 성곽 도시의 어느 외진 돌벽 골목을 걷고 있습니다. "
                       "후드를 깊게 눌러썼음에도 숨겨지지 않는 눈부시게 하얀 머리칼과 백색증 특유의 붉은 눈동자. 낮 동안 마주쳤던 마을 사람들은 당신의 그 이질적이고 아름다운 외모를 보며 '악마의 징표'라 수군대며 침을 뱉곤 했습니다.\n\n"
                       "터덜터덜 걷던 중, 발밑의 웅덩이에 비친 당신의 하얀 얼굴과 붉은 눈이 일순간 거꾸로 뒤틀리며, 머릿속으로 정체 모를 서늘한 속삭임이 스쳐 지나갑니다. "
                       "천계의 감시를 피해 지옥의 깊은 심연에서부터 올라온, 형태 없는 어떤 존재가 주변의 어둠을 매개로 보내는 간접적인 파동입니다.\n\n"
                       "*\"가련하고 아름다운 필멸자여... 저 눈먼 광신도들의 군대 사이에서 네 무력함과 억울함이 느껴지는구나. 그 순백의 육신에 어울리는 힘을 원하느냐?\"*\n\n"
                       "성벽 너머로 밤을 순찰하는 병사들의 갑옷 소리가 멀리서 들려오고, 주변에는 다시 고요한 적막만이 흐릅니다. 당신은 이제 어떻게 행동하겠습니까?"
        }
    ]

for message in st.session_state.messages:
    with st.chat_message(message["role"]):
        st.markdown(message["content"])

# 6. 유저의 행동 처리 (안전한 히스토리 빌드 순서로 최적화)
if user_input := st.chat_input("당신의 행동이나 대사를 입력하세요..."):
    with st.chat_message("user"):
        st.markdown(user_input)
    
    # 영적 연결용 히스토리 목록 사전 구축
    history = []
    for msg in st.session_state.messages:
        role = "user" if msg["role"] == "user" else "model"
        history.append(types.Content(role=role, parts=[types.Part.from_text(text=msg["content"])]))
    
    current_content = types.Content(role="user", parts=[types.Part.from_text(text=user_input)])
    
    # 히스토리 준비가 끝난 후 세션 상태 업데이트
    st.session_state.messages.append({"role": "user", "content": user_input})

    with st.chat_message("assistant"):
        message_placeholder = st.empty()
        full_response = ""
        
        try:
            response_stream = client.models.generate_content_stream(
                model='gemini-2.5-flash',
                contents=history + [current_content],
                config=types.GenerateContentConfig(
                    system_instruction=SYSTEM_INSTRUCTION,
                    temperature=0.8,
                )
            )
            
            for chunk in response_stream:
                full_response += chunk.text
                message_placeholder.markdown(full_response + "▌")
            
            message_placeholder.markdown(full_response)
            
        except Exception as e:
            st.error(f"❌ 차원의 벽이 요동칩니다: {e}")
            full_response = "갑작스러운 영적 간섭으로 인해 주변의 시공간이 일시적으로 얼어붙었습니다. 다시 행동하십시오."
            message_placeholder.markdown(full_response)

    st.session_state.messages.append({"role": "assistant", "content": full_response})
# go