# Projeto Final — Setup Profissional de Python

Esse projeto é a simulação de um ambiente real de engenharia de dados.

A proposta não é só rodar código Python, mas montar um ambiente **reproduzível**: que qualquer pessoa consiga clonar e rodar, com versões do Python controladas, dependências declaradas, credenciais separadas do código e versionamento com Git e GitHub.

> Projeto desenvolvido na Pós-graduação em Engenharia de Dados da **Xperiun**.

---

## 🛠️ Tecnologias e ferramentas

| Ferramenta | Para que serve |
|---|---|
| **pyenv-win** | Controlar várias versões do Python na mesma máquina |
| **venv** | Criar ambientes virtuais isolados por projeto |
| **pipx** | Instalar ferramentas de terminal de forma isolada |
| **uv** | Gerenciar o projeto: ambiente, dependências e execução |
| **python-dotenv** | Ler variáveis de ambiente do arquivo `.env` |
| **pandas** / **requests** | Bibliotecas do projeto |
| **Git** e **GitHub** | Versionamento e publicação do código |

---

## 📁 Estrutura do projeto

```
projeto-final/
├── src/
│   └── projeto_final/
│       └── __init__.py
├── .gitignore         # arquivos que não vão para o repositório (ex.: .env)
├── .python-version    # versão do Python usada no projeto
├── main.py            # script principal
├── pyproject.toml     # nome, versão e dependências do projeto
├── README.md
└── uv.lock            # versões exatas das dependências instaladas
```

> O arquivo `.env` existe apenas localmente e **não** é enviado ao GitHub.

---

## 🚀 Como rodar o projeto

### Pré-requisitos
- [Python](https://www.python.org/) 3.12+
- [uv](https://docs.astral.sh/uv/) instalado (`pip install uv`)
- [Git](https://git-scm.com/)

### Passo a passo

**1. Clone o repositório**
```powershell
git clone https://github.com/MatheusAmaraal/Projeto-Git-Github.git
cd Projeto-Git-Github
```

**2. Instale as dependências**
```powershell
uv sync
```
O `uv` cria o ambiente virtual (`.venv`) e instala exatamente as versões registradas no `uv.lock`.

**3. Crie o seu arquivo `.env`** na raiz do projeto, com as suas credenciais:
```
DB_HOST=seu-servidor
DB_USER=seu-usuario
DB_PASSWORD=sua-senha
API_KEY=sua-chave
```

**4. Execute**
```powershell
uv run main.py
```

Saída esperada:
```
Conectando em servidor.banco.com com usuário admin
Credenciais carregadas com sucesso!
```

---

## 📖 O que foi feito, passo a passo

### 1. Controle de versões do Python com pyenv
Instalação do `pyenv-win` (no PowerShell como Administrador) e de duas versões do Python, que convivem na mesma máquina:
```powershell
pyenv install 3.10.11
pyenv install 3.12.0
pyenv global 3.12.0
```

### 2. Versões diferentes em projetos diferentes
Cada projeto define a própria versão com `pyenv local`, que cria o arquivo `.python-version`:
```powershell
cd projeto-a
pyenv local 3.10.11   # projeto-a usa Python 3.10

cd ../projeto-b
pyenv local 3.12.0    # projeto-b usa Python 3.12
```

### 3. Ambiente virtual com venv
```powershell
python -m venv venv
venv\Scripts\activate
pip install pandas
pip freeze
```
O `pip freeze` mostra vários pacotes para uma única biblioteca instalada, porque o pandas traz as próprias dependências. Com o tempo, fica difícil saber o que foi escolha sua e o que veio junto. Por isso o projeto final usa o `uv`.

### 4. Criação do projeto com uv
```powershell
pip install pipx
pip install uv

uv init projeto-final
cd projeto-final
uv add pandas
uv add requests
uv add python-dotenv
```
Diferente do `pip install`, o `uv add` **declara** a dependência no `pyproject.toml`. Lá aparecem só as bibliotecas que você escolheu, e o `uv.lock` garante as versões exatas para quem clonar.

### 5. Credenciais no `.env`
As credenciais ficam fora do código. O `main.py` só lê as variáveis:
```python
from dotenv import load_dotenv
import os

load_dotenv()

db_host = os.getenv("DB_HOST")
db_user = os.getenv("DB_USER")
db_password = os.getenv("DB_PASSWORD")
api_key = os.getenv("API_KEY")

print(f"Conectando em {db_host} com usuário {db_user}")
print("Credenciais carregadas com sucesso!")
```

### 6. Proteção com `.gitignore`
O `.env` foi adicionado ao `.gitignore`, para que senhas e chaves nunca sejam enviadas ao repositório:
```
# Credenciais
.env
```

### 7. Versionamento e publicação no GitHub
```powershell
git init
git add .gitignore README.md main.py pyproject.toml uv.lock
git commit -m "Salvando meu projeto final e subindo para o GitHub"
git branch -M main
git remote add origin https://github.com/MatheusAmaraal/Projeto-Git-Github.git
git push -u origin main
```

---

## ✅ Boas práticas aplicadas

- **Ambiente reproduzível:** `pyproject.toml` + `uv.lock` permitem recriar o ambiente com um único `uv sync`.
- **Versão do Python fixada** com `.python-version`.
- **Credenciais separadas do código** com `.env` e `python-dotenv`.
- **Dados sensíveis protegidos** pelo `.gitignore`.
- **Commits seletivos:** apenas os arquivos necessários foram enviados.

---

## 👤 Autor

**Matheus Amaral** — Dados e Business Intelligence

[![GitHub](https://img.shields.io/badge/GitHub-MatheusAmaraal-181717?logo=github)](https://github.com/MatheusAmaraal)