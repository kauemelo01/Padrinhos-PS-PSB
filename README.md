# 🎁 Apadrinhamento — Pastoral Social Jardim Colombo

Aplicativo mobile para cadastrar **padrinhos** das crianças atendidas pela Pastoral
Social, construído com **Streamlit**. Lê e grava a planilha `cadastro.xlsx`
hospedada no **Google Drive**, que permanece a fonte da verdade.

> **Projeto derivado do [Cadastro-PS-PSB](https://github.com/kauemelo01/Cadastro-PS-PSB).**
> O Apadrinhamento é um *spin-off* do app principal de cadastro da Pastoral Social:
> usa a mesma planilha, a mesma conta de serviço do Drive e a mesma configuração
> de *Secrets*, mas cuida de uma única tarefa — ligar cada criança a um padrinho.
> Os dois apps podem ficar no ar ao mesmo tempo, gravando no mesmo arquivo
> (veja [Gravação e integridade](#-gravação-e-integridade)).

---

## 🔍 Busca

| Funcionalidade | Descrição |
|---|---|
| **Busca por Nº do registro / carteirinha** | Mostra todas as crianças vinculadas àquela família |
| **Correspondência exata do número** | Zeros à esquerda e casas decimais são ignorados: "015", "15" e "15.0" encontram o mesmo registro |
| **Resumo da família** | Acima dos cartões: Nº, quantidade de crianças e quantas já têm padrinho |

---

## 🧒 Cartão da criança

| Elemento | Descrição |
|---|---|
| **Nome em destaque** | Com sexo e idade logo abaixo |
| **Beneficiário** | Nome do responsável pela cesta, em menor destaque |
| **Observações** | Em destaque amarelo, quando houver |
| **Padrinho** | Nome e telefone em verde quando já cadastrado; "Ainda sem padrinho" quando não |
| **Cor da borda** | Azul para meninos, rosa para meninas |
| **Ícone** | Conforme sexo e faixa etária (tabela abaixo) |

| Faixa | Menino | Menina |
|---|---|---|
| 0 a 3 anos | 👶 🧢 | 👶 🎀 |
| 4 a 8 anos | 👦 | 👧 |
| 9 anos ou mais | 🙋‍♂️ | 🙋‍♀️ |

Idades em meses ("3 meses") contam como 0 anos.

---

## ✏️ Cadastrar / Editar Padrinho

| Funcionalidade | Descrição |
|---|---|
| **Popup por criança** | Botão **➕ Cadastrar Padrinho** (ou **✏️ Editar Padrinho**, se já houver um) |
| **Campos obrigatórios** | Padrinho e Telefone; o telefone precisa de DDD + número (mín. 10 dígitos) |
| **Limpeza automática** | Espaços extras no nome são removidos antes de gravar |
| **Nada mudou, nada grava** | Se os valores são iguais aos atuais, o popup fecha sem ir ao Drive |
| **Confirmação** | Mensagem de sucesso no topo depois que o Drive confirma a gravação |

---

## 📊 Estatísticas

No rodapé, sempre sobre a lista completa (não só a família buscada):

- **Apadrinhadas**, **% concluído**, **total** e **quantas faltam**;
- uma barra de progresso por faixa etária (0 a 3, 4 a 8, 9 anos ou mais);
- aviso de quantas crianças estão sem idade informada — elas entram só no total.

---

## 💾 Gravação e integridade

Toda alteração vai para o `.xlsx` no Google Drive através de uma conta de serviço.
Como cada gravação reenvia o **arquivo inteiro**, e o app de cadastro grava no
mesmo arquivo, o app adota estas salvaguardas:

- **Escrita serializada** — uma trava única no processo cobre todo o ciclo
  baixar → alterar → enviar. Sem ela, dois voluntários salvando ao mesmo tempo
  fariam o envio mais lento apagar o mais rápido, sem erro algum.
- **Checagem de versão no Drive** — a versão do arquivo é conferida logo antes do
  envio. Se outro processo (por exemplo, o app de cadastro) gravou nesse intervalo,
  o ciclo é refeito do zero com o arquivo novo, até 3 tentativas. Se o arquivo
  continuar mudando, nada é gravado e o usuário é avisado para tentar de novo.
- **Concorrência otimista por criança** — o popup lembra o padrinho e o telefone
  que exibia ao abrir. Se outra pessoa alterou essa criança enquanto o formulário
  estava aberto, a gravação é recusada e o app mostra quem está registrado agora.
- **Endereçamento por conteúdo** — a linha é localizada por **Nº + nome da
  criança**, e as colunas pelo nome do cabeçalho, sempre no arquivo recém-baixado.
  Reordenar a planilha ou inserir colunas não faz o padrinho ir para a criança errada.
- **Recusa em caso de ambiguidade** — se a criança não for encontrada, ou houver
  duas com o mesmo nome no mesmo Nº, nada é alterado e o motivo é exibido.
- **Gravação antes da tela** — o cartão só mostra o novo padrinho depois que o
  Drive confirma, para nunca exibir como salvo algo que falhou. A alteração aparece
  na hora, mesmo antes de o cache recarregar o arquivo.
- **Duplo carimbo de tempo (opcional)** — se a aba tiver as colunas
  `Salvo no App` e `Gravado na Planilha`, cada gravação registra quando o botão foi
  clicado e quando chegou à planilha; a diferença revela espera de rede ou de trava.
  As colunas não são criadas automaticamente.

---

## 🗂️ Estrutura da planilha

O app usa sempre a aba **`Lista Ordenada`** de `cadastro.xlsx` (as outras abas,
inclusive as do app de cadastro, são ignoradas). O nome da aba é localizado sem
diferenciar acentos, maiúsculas ou espaços extras. Cabeçalho na linha 1:

| Cabeçalho | Observação |
|---|---|
| `Nº Família / Carteirinha` | obrigatório |
| `Beneficiário (Cesta)` | |
| `Nome da criança` | obrigatório |
| `Sexo` | `Masculino` / `Feminino` |
| `Idade` | ex.: `6 anos`, `3 meses` |
| `Observações` | |
| `Padrinho` | criada automaticamente no 1º salvamento |
| `Telefone Padrinho` | criada automaticamente no 1º salvamento |
| `Salvo no App` | opcional — carimbo de tempo do clique |
| `Gravado na Planilha` | opcional — carimbo de tempo da gravação |

Notas:
- A ordem das colunas não importa; cada uma é localizada pelo nome do cabeçalho
  (ignorando acentos e maiúsculas).
- Linhas sem número ou sem nome da criança são ignoradas.
- Carimbos de tempo usam o formato `dd/mm/aaaa hh:mm:ss`.

---

## ⚙️ Configuração

A configuração é a mesma do [app de cadastro](https://github.com/kauemelo01/Cadastro-PS-PSB):
se ele já está no ar, basta reaproveitar os mesmos *Secrets*.

### 1. Dependências

`requirements.txt`:

```
streamlit>=1.50
pandas
openpyxl
requests
google-api-python-client
google-auth
```

### 2. Google Drive

1. Crie uma *service account* no Google Cloud com a **Google Drive API** ativada e gere uma chave JSON.
2. Compartilhe o arquivo `cadastro.xlsx` no Drive com o e-mail da service account, com permissão de **Editor**.
3. Gere um link de download direto do arquivo (formato `https://drive.google.com/uc?export=download&id=<ID_DO_ARQUIVO>`).

### 3. Secrets

No Streamlit Community Cloud (*App → Settings → Secrets*) ou em `.streamlit/secrets.toml` para rodar localmente:

```toml
GITHUB_RAW_URL = "https://drive.google.com/uc?export=download&id=<ID_DO_ARQUIVO>"
GDRIVE_FILE_ID = "<ID_DO_ARQUIVO>"
GDRIVE_SA_CREDENTIALS = '''
{ ...conteúdo do JSON da service account... }
'''
```

| Secret | Uso |
|---|---|
| `GITHUB_RAW_URL` | link de leitura da planilha (apesar do nome, aceita o link do Drive) |
| `GDRIVE_FILE_ID` | ID do arquivo no Drive, usado para gravar |
| `GDRIVE_SA_CREDENTIALS` | credenciais da service account (JSON) |

Sem `GDRIVE_FILE_ID` e `GDRIVE_SA_CREDENTIALS`, o app funciona só para consulta e
exibe um aviso de que o salvamento está desativado.

> ⚠️ Nunca faça commit do `secrets.toml` nem do JSON da service account. Adicione
> `.streamlit/secrets.toml` ao `.gitignore`.

---

## 🚀 Rodando localmente

```bash
pip install -r requirements.txt
streamlit run main.py
```

## ☁️ Deploy

1. Suba `main.py` e `requirements.txt` para este repositório.
2. Em [share.streamlit.io](https://share.streamlit.io), crie um app apontando para o repositório, branch `main` e arquivo `main.py`.
3. Configure os Secrets conforme acima.

---

## 🛠️ Observações técnicas

- Os dados ficam em cache por 5 minutos; após cada gravação o cache é limpo.
- O download trata a página de confirmação que o Google Drive exibe para arquivos grandes.
- Ao procurar a linha da criança, o app lê só as células preenchidas, para não varrer
  milhares de linhas vazias quando a planilha tem formatação até o fim da folha.
- A trava de escrita vale dentro de um processo. Entre processos diferentes (dois
  apps, ou várias instâncias), quem protege é a checagem de versão no Drive.
- Layout mobile-first, com paleta própria para o modo escuro.

## 📄 Licença

[GNU General Public License v3](LICENSE).
