# URL Shortener - AWS Lambda

Este projeto implementa um serviço de encurtador de URLs utilizando AWS Lambda e S3. Ele é composto por duas funções Lambda, cada uma em um projeto separado:

1. **Create URL Shortener**: Recebe uma URL longa e um tempo de expiração e retorna um código curto associado à URL.
2. **Redirect URL Shortener**: Recebe um código curto e redireciona o usuário para a URL longa, se ainda estiver válida.

---

## **Requisitos**

- **Java 17 ou superior**
- **Maven**
- Dependências:
  - AWS SDK v2 para Java
  - Jackson (para serialização/deserialização JSON)
  - Lombok (para getters e setters)
- **Conta AWS configurada com permissões para Lambda e S3**
- Um bucket S3 para armazenar as URLs encurtadas.

---

## **Configuração do Ambiente**

1. **Configuração do Bucket S3**:
   - Crie um bucket no S3, como `gama-url-shortener-storage`.
   - Certifique-se de que o bucket está acessível para leitura/escrita pela função Lambda.

2. **Configuração do Lombok**:
   - Certifique-se de que o plugin Lombok está instalado na sua IDE.
   - Habilite o processamento de anotações.

3. **Dependências no `pom.xml`**:

```xml
<dependencies>
    <dependency>
        <groupId>software.amazon.awssdk</groupId>
        <artifactId>s3</artifactId>
        <version>2.20.40</version> <!-- Verifique a versão mais recente -->
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.15.2</version>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.28</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

---

## **Arquitetura**

O projeto está dividido em duas funções:

### 1. **Create URL Shortener**
- **Objetivo**: Gerar um código curto para uma URL longa.
- **Processo**:
  - Recebe a URL longa e um tempo de expiração em segundos no corpo da requisição.
  - Gera um código único (`UUID`) para a URL.
  - Armazena os dados (URL longa e expiração) em um arquivo JSON no S3.
  - Retorna o código curto ao cliente.

#### Exemplo de Requisição:
```json
POST /create
{
  "originalUrl": "https://example.com/long-url",
  "expirationTime": "3600" // Em segundos
}
```

#### Exemplo de Resposta:
```json
{
  "code": "abcd1234"
}
```

### 2. **Redirect URL Shortener**
- **Objetivo**: Redirecionar o código curto para a URL longa associada.
- **Processo**:
  - Recebe um código curto como parte do caminho (`/abcd1234`).
  - Busca os dados correspondentes no S3.
  - Verifica se o tempo de expiração ainda é válido.
  - Retorna um redirecionamento (HTTP 302) ou erro (HTTP 410).

#### Exemplo de Resposta:
- **URL válida**:
  - Status: `302 Found`
  - Headers:
    ```http
    Location: https://example.com/long-url
    ```
- **URL expirada**:
  - Status: `410 Gone`
  - Body: `"This URL has expired"`

---

## **Configuração do AWS Lambda**

1. **Empacotamento do Projeto**:
   - Use o Maven para criar o arquivo `.jar`:
     ```bash
     mvn clean package
     ```
   - O arquivo gerado estará em `target/`.

2. **Deploy para AWS Lambda**:
   - Crie duas funções Lambda:
     - **Create URL Shortener**
       - Configuração do handler: `com.rocketseat.createUrlShortner.Main::handleRequest`
     - **Redirect URL Shortener**
       - Configuração do handler: `org.rocketseat.redirectUrlShortner.Main::handleRequest`
   - Configure a variável de ambiente com o nome do bucket S3 (se necessário).

3. **Configuração do Gateway API**:
   - Configure o Gateway API para encaminhar as requisições para as funções Lambda:
     - **/create** → Lambda de criação
     - **/{code}** → Lambda de redirecionamento

---

## **Testes**

### Local
Use uma ferramenta como `curl` ou um cliente REST (Postman) para testar localmente antes de fazer o deploy.

### AWS
Após o deploy, teste as URLs públicas do Gateway API para verificar o funcionamento.

---

## **Futuras Melhorias**

- Implementação de autenticação para gerenciar URLs.
- Suporte a customização de códigos curtos.
- Integração com banco de dados (DynamoDB) para escalabilidade.

---

## **Contribuições**

Sinta-se à vontade para enviar *issues* e *pull requests* para este repositório. Todos os tipos de contribuição são bem-vindos!
