# resposta-ao-desafio-DIO-valida-o-de-cartao-de-credito
resposta ao desafio DIO validação de cartao de credito
Para criar um aplicativo Streamlit que permite o upload de uma imagem de cartão de crédito e utiliza o Azure Document Intelligence (anteriormente Azure Form Recognizer) para validar o cartão, você pode seguir estas etapas (fiz um pouco diferente do que o video)

### Requisitos
1. **Azure Account**: Configure o Azure Document Intelligence no portal do Azure e obtenha as chaves e o endpoint.
2. **Bibliotecas Necessárias**:
   - `streamlit`
   - `requests` (para fazer chamadas à API do Azure)
   - `Pillow` (para processar imagens, opcional)

Instale as bibliotecas necessárias com:
```bash
pip install streamlit requests pillow
```

---

### Código do Aplicativo

```python
import streamlit as st
import requests
from PIL import Image
import io

# Configurações da API do Azure
AZURE_ENDPOINT = "https://<seu-endpoint>.cognitiveservices.azure.com/"
AZURE_API_KEY = "<sua-chave-da-api>"
MODEL_ID = "prebuilt-idDocument"  # Model ID para documentos de identidade (ou outro apropriado)

def analyze_credit_card(image_data):
    """
    Envia a imagem para o Azure Document Intelligence e retorna a análise.
    """
    url = f"{AZURE_ENDPOINT}formrecognizer/documentModels/{MODEL_ID}:analyze?api-version=2023-07-31"
    headers = {
        "Ocp-Apim-Subscription-Key": AZURE_API_KEY,
        "Content-Type": "image/jpeg",
    }

    response = requests.post(url, headers=headers, data=image_data)
    if response.status_code == 200:
        return response.json()
    else:
        st.error(f"Erro na API: {response.status_code} - {response.text}")
        return None

# Interface do Streamlit
st.title("Validação de Cartão de Crédito com Azure Document Intelligence")

uploaded_file = st.file_uploader("Envie uma imagem do cartão de crédito", type=["jpg", "jpeg", "png"])

if uploaded_file:
    try:
        # Carregar e exibir a imagem
        image = Image.open(uploaded_file)
        st.image(image, caption="Imagem carregada", use_column_width=True)

        # Converter a imagem em bytes
        img_byte_arr = io.BytesIO()
        image.save(img_byte_arr, format='JPEG')
        img_byte_arr = img_byte_arr.getvalue()

        # Analisar a imagem usando o Azure Document Intelligence
        with st.spinner("Analisando a imagem..."):
            result = analyze_credit_card(img_byte_arr)

        # Exibir resultados
        if result:
            st.subheader("Resultado da análise:")
            st.json(result)

            # Extração de informações úteis
            st.subheader("Informações Extraídas:")
            fields = result.get("fields", {})
            for field, details in fields.items():
                value = details.get("value", "N/A")
                confidence = details.get("confidence", 0)
                st.write(f"**{field}**: {value} (Confiança: {confidence * 100:.2f}%)")
    except Exception as e:
        st.error(f"Ocorreu um erro: {e}")
```

---

### Passos para Configurar e Executar
1. Substitua `AZURE_ENDPOINT` e `AZURE_API_KEY` pelas suas informações da conta do Azure.
2. Verifique se o `MODEL_ID` corresponde ao modelo correto para processar cartões ou documentos relacionados.
3. Salve o código em um arquivo `app.py`.
4. Execute o aplicativo Streamlit:
   ```bash
   streamlit run app.py
   ```

---

### Funcionalidades Adicionais (opcionais)
- **Validação Personalizada**: Adicione lógica para validar o número do cartão de crédito (usando o algoritmo Luhn, por exemplo).
- **Segurança**: Certifique-se de que os dados confidenciais não são armazenados ou compartilhados.
- **Outros Modelos**: Caso você precise de um modelo customizado para cartões de crédito, treine e publique um no Azure.

Essa aplicação básica valida e exibe informações extraídas do cartão com alta flexibilidade!
