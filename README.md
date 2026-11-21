# 🎁 Apadrinhamento — Pastoral Social Jardim Colombo

App web (Streamlit), pensado para celular, para cadastrar padrinhos das crianças atendidas pela Pastoral Social. Os dados são lidos e gravados diretamente na planilha `cadastro.xlsx` no Google Drive.

## Funcionalidades

- **Busca por Nº do registro / carteirinha**: mostra todas as crianças vinculadas àquela família.
- **Cartão por criança** com:
  - nome da criança em destaque, sexo e idade;
  - nome do beneficiário (em menor destaque);
  - observações da criança, quando houver;
  - ícone conforme sexo e faixa etária:

    | Faixa         | Menino   | Menina   |
    |---------------|----------|----------|
    | 0 a 3 anos    | 👶 🧢    | 👶 🎀    |
    | 4 a 8 anos    | 👦       | 👧       |
    | 9 anos ou mais| 🙋‍♂️      | 🙋‍♀️      |

- **Cadastrar / Editar Padrinho**: botão em cada criança abre um popup com os campos **Padrinho** e **Telefone** (ambos obrigatórios; telefone com mín. 9 dígitos, DDD opcional). Se a criança já tem padrinho, ele é exibido no cartão e o botão vira "Editar Padrinho".
- **Estatísticas** no rodapé: crianças apadrinhadas, % concluído, total e quantas faltam — no geral e por faixa etária (0 a 3, 4 a 8, 9 ou mais).

## Estrutura da planilha

O app usa sempre a aba **`Lista Ordenada`** de `cadastro.xlsx` (outras abas são ignoradas), com cabeçalho na linha 1:

| Coluna | Cabeçalho                  | Observação                                   |
|--------|----------------------------|----------------------------------------------|
| A      | `Nº Família / Carteirinha` | obrigatório                                  |
| B      | `Beneficiário (Cesta)`     |                                              |
| C      | `Nome da criança`          | obrigatório                                  |
| D      | `Sexo`                     | `Masculino` / `Feminino`                     |
| E      | `Idade`                    | ex.: `6 anos`, `3 meses`                     |
| F      | `Observações`              |                                              |
| —      | `Padrinho`                 | criada automaticamente no 1º salvamento      |
| —      | `Telefone Padrinho`        | criada automaticamente no 1º salvamento      |
| —      | `Salvo no App`              | criada automaticamente no 1º salvamento      |
| —      | `Gravado na Planilha`       | criada automaticamente no 1º salvamento      |

Notas:
- A ordem das colunas não importa; o app localiza cada uma pelo nome do cabeçalho (ignorando acentos e maiúsculas).
- Linhas sem número ou sem nome da criança são ignoradas.
- Ao salvar, a linha é localizada pelo **Nº + nome da criança**, então reordenar a planilha não faz o padrinho ser gravado na criança errada.

## Carimbos de tempo

Toda gravação registra dois horários, na própria linha da criança:

- **Salvo no App**: o momento em que o voluntário clicou em salvar.
- **Gravado na Planilha**: o momento em que a gravação realmente chegou ao arquivo no Drive.

As duas datas costumam diferir por poucos segundos — o tempo de baixar o arquivo, alterar a célula e reenviar. Uma diferença maior, ou vários registros em fila no mesmo minuto, indicam vários voluntários salvando ao mesmo tempo: as gravações são serializadas (uma de cada vez) para que nenhuma seja perdida, então quem está na fila espera um pouco mais.

As colunas `Salvo no App` e `Gravado na Planilha` são criadas automaticamente, com o mesmo estilo do restante do cabeçalho, na primeira vez que alguém salva um padrinho — não é preciso criá-las manualmente. Se elas já existirem na aba (por exemplo, criadas à mão ou por uma gravação anterior), o app reaproveita as mesmas colunas, não duplica.

Os horários são gravados no fuso de Brasília, independentemente de onde o servidor do app estiver rodando.

## Configuração

### 1. Dependências

`requirements.txt`:

```
streamlit
pandas
openpyxl
requests
google-auth
google-api-python-client
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

| Secret                  | Uso                                              |
|-------------------------|--------------------------------------------------|
| `GITHUB_RAW_URL`        | link de leitura da planilha                      |
| `GDRIVE_FILE_ID`        | ID do arquivo no Drive, usado para gravar        |
| `GDRIVE_SA_CREDENTIALS` | credenciais da service account (JSON)            |

Sem `GDRIVE_FILE_ID` e `GDRIVE_SA_CREDENTIALS`, o app funciona só para consulta e exibe um aviso de que o salvamento está desativado.

> ⚠️ Nunca faça commit do `secrets.toml` nem do JSON da service account. Adicione `.streamlit/secrets.toml` ao `.gitignore`.

## Rodando localmente

```bash
pip install -r requirements.txt
streamlit run main.py
```

## Deploy

1. Suba `main.py` e `requirements.txt` para este repositório.
2. Em [share.streamlit.io](https://share.streamlit.io), crie um app apontando para o repositório, branch `main` e arquivo `main.py`.
3. Configure os Secrets conforme acima.

## Observações técnicas

- Os dados ficam em cache por 5 minutos; após cada salvamento o cache é limpo e a alteração aparece na hora.
- Cada gravação baixa a versão mais recente do arquivo no Drive, altera apenas as células da criança (padrinho, telefone e os dois carimbos de tempo) e reenvia o arquivo.
- As gravações são serializadas por um lock interno do app: mesmo com vários voluntários salvando ao mesmo tempo, cada ciclo de baixar/alterar/reenviar acontece por vez, sem sobrescrever o trabalho de outra pessoa.
- Antes de salvar, o app confere se o padrinho da criança não foi alterado por outra pessoa enquanto o formulário estava aberto; se foi, a gravação é recusada com um aviso, em vez de sobrescrever silenciosamente.
