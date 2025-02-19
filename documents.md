## Sistema de Recomendação de Imagens com Redes Neurais

Este projeto demonstra como utilizar um modelo pré-treinado (ResNet50) para extrair características (features) de imagens e recomendar imagens similares com base na similaridade cosseno. O script integra a busca por imagens na web (usando o Google Imagens), o upload interativo da imagem de referência (via Google Colab) e a exibição dos resultados.

### Funcionalidades

- **Upload de Imagem:** Permite que o usuário faça upload de uma imagem de referência.
- **Extração de Features:** Utiliza a ResNet50 (com a última camada removida) para extrair embeddings da imagem.
- **Busca de Imagens:** Realiza scraping no Google Imagens para coletar URLs de imagens relacionadas à query "tênis".
- **Cálculo de Similaridade:** Compara o embedding da imagem de referência com os embeddings das imagens candidatas usando a similaridade cosseno.
- **Exibição de Resultados:** Mostra, lado a lado, a imagem de referência e as imagens recomendadas.

## Dependências

Certifique-se de ter as seguintes bibliotecas instaladas:

- Python 3.x
- requests
- numpy
- torch
- torchvision
- Pillow
- scikit-learn
- beautifulsoup4
- matplotlib
- google.colab (caso use o Google Colab)

### **Observação: O código foi desenvolvido e testado no Google Colab, aproveitando o widget de upload disponível na plataforma.**

## Como Utilizar

**1. Configuração do Ambiente:**

- Recomenda-se utilizar o [Google Colab](colab.new) para executar o código, pois ele facilita o upload de arquivos.
- Caso execute localmente, adapte a parte de upload (por exemplo, utilizando um seletor de arquivos do seu ambiente).

**2. Execução do Script:**

- Faça o upload do código no ambiente de execução (por exemplo, em um notebook Colab).
- Execute as células do notebook.
- Quando solicitado, faça o upload da imagem de referência (por exemplo, uma foto de um tênis).
- O script realizará a busca de imagens com a query "tênis", extrairá os embeddings e exibirá a imagem de referência juntamente com 4 imagens similares.

**3. Personalizações Possíveis:**

- **Query de Busca:** Atualmente, a função search_images realiza uma busca por "tênis". Você pode alterar a URL para buscar outras categorias ou produtos.
- **Número de Resultados:** Altere o parâmetro num_results da função search_images para aumentar ou diminuir a quantidade de imagens buscadas.
- **Número de Recomendações:** Modifique o parâmetro top_n na função find_similar_images para definir quantas imagens similares serão exibidas.

## Estrutura do Código
1. Busca de Imagens
python
Copy
Edit
def search_images(num_results=10):
    search_url = "https://www.google.com/search?q=tênis&tbm=isch"
    headers = {"User-Agent": "Mozilla/5.0"}
    response = requests.get(search_url, headers=headers)
    soup = BeautifulSoup(response.text, "html.parser")
    image_elements = soup.find_all("img", limit=num_results+1)
    image_urls = [img["src"] for img in image_elements[1:]]  # Ignora o primeiro resultado (possivelmente um ícone)
    return image_urls
   
- Objetivo: Coletar URLs de imagens a partir de uma busca no Google Imagens.
- Parâmetro: num_results define quantas imagens serão coletadas (padrão 10).

2. Preparação do Modelo e Transformação da Imagem
Modelo: Utiliza a ResNet50 pré-treinada do PyTorch, removendo a última camada (Fully Connected) para obter embeddings.
Transformação: A imagem é redimensionada para 224x224, convertida para tensor e normalizada de acordo com os valores do ImageNet.
python
Copy
Edit
model = models.resnet50(pretrained=True)
model = torch.nn.Sequential(*(list(model.children())[:-1]))  # Remove a última camada FC
model.eval()

transform = transforms.Compose([
    transforms.Resize((224, 224)),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225])
])

3. Extração do Embedding
python
Copy
Edit
def get_image_embedding(image):
    image = image.convert('RGB')
    image = transform(image).unsqueeze(0)
    with torch.no_grad():
        embedding = model(image).squeeze().numpy()
    return embedding.flatten()
Objetivo: Converter uma imagem em um vetor de características (embedding) usando a ResNet50.
4. Comparação e Seleção de Imagens Similares
python
Copy
Edit
def find_similar_images(reference_embedding, candidate_urls, top_n=4):
    candidate_embeddings = [get_image_embedding(Image.open(BytesIO(requests.get(url).content))) for url in candidate_urls]
    similarities = cosine_similarity([reference_embedding], candidate_embeddings)[0]
    ranked_urls = [url for _, url in sorted(zip(similarities, candidate_urls), reverse=True)][:top_n]
    return ranked_urls
Objetivo: Calcular a similaridade entre a imagem de referência e as imagens candidatas, selecionando as top_n mais similares.<br>
5. Exibição das Imagens
python
Copy
Edit
def display_images(reference_image, similar_images):
    fig, axes = plt.subplots(1, 5, figsize=(20, 5))

    # Exibe a imagem de referência
    axes[0].imshow(reference_image)
    axes[0].set_title("Quero Comprar")
    axes[0].axis("off")

    # Exibe as imagens recomendadas
    for i, img_url in enumerate(similar_images):
        img = Image.open(BytesIO(requests.get(img_url).content))
        axes[i+1].imshow(img)
        axes[i+1].set_title("Recomendações")
        axes[i+1].axis("off")

    plt.show()
Objetivo: Exibir a imagem de referência e as imagens recomendadas em um layout lado a lado.
6. Fluxo Principal do Script
O script solicita o upload da imagem de referência.
Extrai o embedding da imagem.
Busca imagens relacionadas à query "tênis".
Encontra e exibe as imagens mais similares.
python
Copy
Edit
# Upload de imagem pelo usuário (utilizando Google Colab)
uploaded = files.upload()
image_path = list(uploaded.keys())[0]
reference_image = Image.open(image_path)
reference_embedding = get_image_embedding(reference_image)

# Busca de imagens similares
image_urls = search_images()

if image_urls:
    similar_images = find_similar_images(reference_embedding, image_urls, top_n=4)
    display_images(reference_image, similar_images)
else:
    print("Nenhuma imagem encontrada.")
    
## Considerações Finais
Depreciações: O uso do parâmetro pretrained=True pode gerar avisos de depreciação. Considere atualizar para o parâmetro weights conforme as recomendações das versões mais recentes do torchvision.
Scraping do Google: A extração de imagens via scraping pode ser sensível a mudanças na estrutura da página do Google Imagens e a restrições de acesso.

## Contribuições
Contribuições são bem-vindas! Caso identifique problemas ou tenha sugestões de melhorias, sinta-se à vontade para abrir uma issue ou enviar um pull request.

## Licença
Este projeto é licenciado sob a MIT License.
