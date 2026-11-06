# 🛡️ Sistema de Controle Financeiro & Quitação de Dívidas PF (Sheets + GitHub Pages)

Aplicação web responsiva e **100% gratuita (Serverless)** projetada para reorganização financeira pessoal, controle de múltiplos cartões de crédito e estratégia de quitação de dívidas (Pessoas Físicas e Instituições). Desenvolvida com arquitetura **White-Label (Zero Hardcoding)** para implantação rápida em qualquer conta Google.

---

## 🏗️ Arquitetura do Sistema

1. **Banco de Dados (Google Sheets):** Armazena o fluxo mensal, matriz de custos fixos/rendas, configuração e faturas de até 4 cartões de crédito, além do mapa de passivos e histórico de amortizações em 7 abas estruturadas.
2. **Backend / API (Google Apps Script):** Script vinculado à planilha (`doGet` e `doPost`) com travas de concorrência (`LockService`), cálculo automático de fechamento de faturas, preenchimento automático de juros de mercado/SELIC e validação via Token/PIN.
3. **Frontend (GitHub Pages):** Single-Page Application (SPA) em arquivo único (`index.html`), utilizando **Tailwind CSS**, **Chart.js** e **FontAwesome**. Credenciais isoladas no `localStorage` (`fin_pf_api_url` e `fin_pf_api_token`), permitindo uso simultâneo com o sistema PJ no mesmo navegador sem conflito.

---

## ✨ Funcionalidades Principais

* **1. Dash Principal (Visão Integrada do Mês):**
  * **Resumo Financeiro Unificado:** Calcula automaticamente `Renda do Mês - (Custos Fixos + Faturas dos 4 Cartões + Parcelas de Acordos do Mês)` para exibir o **Fôlego Livre no Mês**.
  * **Termômetro Global de Quitação:** Barra de progresso do passivo total, percentual quitado e soma de descontos reais conquistados em renegociações.
  * **Ordem de Ataque Recomendada:** Alternância em tempo real entre o **Método Avalanche** (foco nas dívidas com maiores taxas de juros) e o **Método Bola de Neve** (foco nos menores saldos devedores para ganhar tração rápida).
* **2. Gestão de Cartões de Crédito (Até 4 Cartões Independentes):**
  * Configuração individual de Nome, Cor, Limite Total, Dia de Fechamento e Dia de Vencimento.
  * **Cálculo Inteligente de Fatura:** Ao lançar uma compra à vista ou parcelada, o sistema verifica a data da compra contra o Dia de Fechamento daquele cartão e distribui automaticamente as parcelas nos meses futuros corretos.
  * **Assinaturas Recorrentes no Cartão:** Compras marcadas como assinatura fixa são replicadas automaticamente a cada novo mês.
* **3. Custos Fixos & Renda Mensal:**
  * Matriz de salários/rendas e despesas fixas da residência com botão **"Gerar Fixos + Assinaturas no Mês"** (com trava contra duplicidade).
* **4. Mapa de Dívidas & Acordos (PF vs. Instituições):**
  * **Dívidas com Pessoas Físicas:** Taxa travada em **0% de juros** e amortização livre (abatimentos variáveis conforme disponibilidade de caixa).
  * **Dívidas com Instituições:** Preenchimento automático da taxa de juros mensal (SELIC / Mercado) conforme a modalidade financeira quando o usuário não declara a taxa.
  * **Modo Acordo Fechado:** Calcula o desconto obtido frente ao valor atualizado e gera automaticamente o cronograma de parcelas mensais.
* **Trava Anti-Duplicidade:** Bloqueio imediato de botões (`disabled` + spinner) no primeiro clique/toque e controle de fila no backend.

---

## 📊 Estrutura do Banco de Dados (Abas da Planilha PF)

| Aba | Colunas | Finalidade |
| :--- | :--- | :--- |
| **`Fluxo_Mes`** | `ID`, `Mes_Ano`, `Data_Vencimento`, `Data_Pagamento`, `Tipo`, `Categoria`, `Descricao`, `Valor`, `Status`, `Origem_Ref`, `Observacoes` | Lançamentos de entradas e saídas de caixa do mês. |
| **`Custos_Fixos`** | `ID_Fixo`, `Tipo`, `Descricao`, `Categoria`, `Valor_Base`, `Dia_Vencimento`, `Ativo` | Matriz de rendas e despesas fixas mensais. |
| **`Cartoes_Config`** | `ID_Cartao`, `Nome_Cartao`, `Cor_Hex`, `Limite_Total`, `Dia_Fechamento`, `Dia_Vencimento`, `Ativo` | Parâmetros dos 4 slots de cartões de crédito. |
| **`Cartoes_Lancamentos`** | `ID_Compra`, `ID_Grupo_Parcela`, `ID_Cartao`, `Data_Compra`, `Mes_Fatura`, `Descricao`, `Categoria`, `Valor_Parcela`, `Parcela_Atual`, `Total_Parcelas`, `Assinatura_Fixa`, `Status_Fatura` | Parcelas e assinaturas alocadas por fatura mensal. |
| **`Dividas`** | `ID_Divida`, `Tipo_Credor`, `Credor`, `Descricao`, `Modalidade`, `Data_Inicio`, `Valor_Original`, `Taxa_Juros_Mes`, `Valor_Atualizado`, `Em_Acordo`, `Valor_Acordo`, `Desconto_Obtido`, `Qtd_Parcelas`, `Parcelas_Pagas`, `Valor_Pago_Total`, `Saldo_Restante`, `Dia_Venc_Acordo`, `Prioridade`, `Status`, `Observacoes` | Cadastro mestre de passivos PF e Instituições. |
| **`Dividas_Pagamentos`** | `ID_Pagto`, `ID_Divida`, `Data_Vencimento`, `Data_Pagamento`, `Numero_Parcela`, `Valor_Previsto`, `Valor_Pago`, `Status`, `Observacoes` | Cronograma de parcelas de acordos e histórico de amortizações avulsas. |
| **`Config`** | `Chave`, `Valor`, `Descricao` | Parâmetros globais (`API_TOKEN`, `NOME_TITULAR`, `TAXA_SELIC_MENSAL`). |

---

## 🚀 Guia de Implantação Rápida

### 1. Planilha e Backend (Google Sheets)
1. Crie uma **Planilha em branco** na conta Google destinada ao controle PF.
2. Acesse **Extensões** > **Apps Script**, cole o conteúdo de `Codigo.gs` e salve.
3. Selecione a função **`setupInicialPF`** na barra superior e clique em **Executar** (autorize as permissões).
4. Na aba **`Config`** criada na planilha, defina o seu **`API_TOKEN`** (PIN de segurança) e o **`NOME_TITULAR`**.

### 2. Publicação do Web App (API)
1. No Apps Script, clique em **Implantar** > **Nova implantação** > tipo **App da Web**.
2. Configure:
   * **Executar como:** `Eu`
   * **Quem pode acessar:** `Qualquer pessoa`
3. Clique em **Implantar** e copie a URL terminada em `/exec`.

### 3. Publicação no GitHub Pages
1. Suba o arquivo `index.html` neste repositório e ative o **GitHub Pages** em *Settings > Pages > Branch: main*.
2. Acesse o link gerado, clique no ícone de **Engrenagem ⚙️** e insira a URL do Web App e o PIN configurado.
