# Meu Controle

Dashboard pessoal de **produtividade no trabalho** e **controle financeiro**: salário, produtividade (conferência e separação), despesas, dívidas, livro-caixa, planejamento mensal e **relatórios em PDF A4**.

Tudo roda no navegador, sem servidor e sem API externa. Interface em português do Brasil.

## Recursos

| Módulo | O que faz |
| --- | --- |
| **Início** | Cards, gráficos (receitas × despesas, saldo, categorias, produtividade diária), contas a vencer, dívidas próximas, últimas movimentações, % do salário comprometido, filtros por mês, ano e período personalizado. |
| **Produtividade** | Registros de conferência/separação (vários por dia), histórico por dia/semana/mês, média, melhor dia, comparação com o período anterior, pagamento integral/parcial/pendente, exportação CSV e PDF. |
| **Salário** | Detalhamento transparente (base, adicional noturno por horas realizadas, assiduidade, produtividade, extras, descontos), parâmetros por mês, controle de pagamentos e diferença entre estimado e recebido. |
| **Despesas** | Única, semanal, mensal, anual e parcelada; recorrências geradas sem duplicar; pagar/desfazer; filtros; planejado × realizado. |
| **Dívidas** | Saldo, parcelas, % quitado, pagamentos parciais/totais, previsão de quitação e simulador de pagamentos extras. |
| **Livro-caixa** | Entradas e saídas, saldo realizado × previsto, contas, transferências, lançamentos vinculados a salário/despesa/dívida sem contagem dupla. |
| **Planejamento** | Compromissos do mês, saldo previsto, meta de economia, orçamento por categoria, alertas, metas financeiras. |
| **Relatórios** | 8 relatórios em PDF A4 com pré-visualização, download e impressão. |
| **Configurações** | Todos os valores editáveis, categorias, contas, orçamentos, aparência, backup JSON, CSV, restaurar e apagar dados. |

## Requisitos

- [Node.js](https://nodejs.org) 18 ou superior

## Instalar e executar

```bash
npm install
npm run dev        # abre em http://localhost:5173
```

Outros comandos:

```bash
npm test           # testes dos cálculos e dos relatórios
npm run typecheck  # verificação de tipos
npm run build      # gera a pasta dist/ para publicar
npm run preview    # serve a pasta dist/ localmente
```

## Publicar no GitHub Pages

1. Crie um repositório no GitHub e envie o projeto:

   ```bash
   git init
   git add .
   git commit -m "Meu Controle"
   git branch -M main
   git remote add origin https://github.com/SEU-USUARIO/meu-controle.git
   git push -u origin main
   ```

2. No GitHub, abra **Settings → Pages** e em **Source** escolha **GitHub Actions**.
3. O workflow `.github/workflows/deploy.yml` roda os testes, gera o build e publica a cada push na branch `main`.
4. O site ficará em `https://SEU-USUARIO.github.io/meu-controle/`.

O Vite usa `base: './'` e a navegação usa `#/rota`, por isso o projeto funciona em qualquer subcaminho de hospedagem estática (Netlify, Vercel, Cloudflare Pages, servidor próprio etc.), sem configuração extra.

## Login, senha e segurança dos dados

Ao abrir o site aparece uma tela de **usuário e senha**. Na primeira vez, crie o acesso; se já havia dados salvos neste navegador (versão anterior, sem senha), eles são trancados com a senha na criação da conta.

- **Cada usuário tem seus próprios dados**, separados e criptografados (AES-GCM 256 bits). A chave é derivada da senha com PBKDF2-SHA256 (600 mil iterações) e fica só na memória; a senha nunca é gravada.
- Nomes de usuário não são gravados em texto, só um hash deles.
- **Bloqueio automático** após 15 minutos sem uso e botão **Sair** no menu. Depois de 5 senhas erradas seguidas, o login passa a esperar um tempo crescente.
- Em **Configurações → Dados e backup → Acesso e senha** é possível trocar a senha e excluir a conta.
- **Não existe "esqueci a senha"**: sem ela os dados não podem ser abertos por ninguém. Guarde a senha.
- O **backup JSON/CSV exportado não é criptografado**; guarde esses arquivos com cuidado.
- O site precisa ser aberto por **https://** (GitHub Pages já é) ou localhost, pois a criptografia do navegador só funciona nesses endereços.

### O que isto protege (e o que não protege)

O app não tem servidor: o site publicado contém só o código, e os dados ficam no LocalStorage de cada navegador. Por isso uma pessoa que acessa o endereço do site **não vê** os dados de outra: cada navegador tem os seus. A senha protege contra quem usa o **mesmo computador/navegador**, e a criptografia protege os dados mesmo se alguém abrir as ferramentas do navegador ou copiar o armazenamento. Uma senha curta ou comum continua sendo adivinhável por tentativa em massa fora do site; use uma senha longa.

Se um dia for preciso acessar os mesmos dados de vários aparelhos ou ter contas "de verdade" na nuvem, é necessário um servidor (por exemplo Supabase ou Firebase) com autenticação própria.

## Onde ficam os dados

Os dados ficam **somente no LocalStorage deste navegador**, neste computador, criptografados com a senha do usuário.

- Não há sincronização entre dispositivos.
- Limpar os dados do site, trocar de navegador ou usar janela anônima faz você perder as informações.
- **Faça backups regulares** em *Configurações → Dados e backup → Backup completo (JSON)* e restaure pelo mesmo caminho. O arquivo é validado antes de ser aplicado.

## Como os cálculos funcionam

- **Produção** = quantidade × valor unitário. Valores monetários são arredondados a 2 casas em um único ponto (`round2`).
- **Total bruto** = salário-base + adicional noturno + assiduidade + produtividade elegível + horas extras + outros adicionais.
- **Líquido estimado** = bruto − descontos (vale-transporte, INSS informado, adiantamentos, outros).
- **Adicional noturno** = valor da hora normal × % × horas noturnas realizadas (horas/dia × dias trabalhados, ou o valor informado para o mês). Não é aplicado sobre o salário inteiro. Há opção de hora noturna reduzida.
- **INSS e descontos legais** não são calculados com alíquotas embutidas: você informa valor ou percentual conforme o holerite; sem isso, aparece como "a confirmar".
- **Estimativa × folha confirmada**: ao registrar o pagamento como recebido/ajustado, ele passa a valer no lugar da estimativa e a diferença é exibida.
- **Sem contagem dupla**: produtividade elegível já está no salário (não é receita extra); pagamento de dívida é uma única saída (não é também despesa); transferências não são receita nem despesa; lançamentos "já no salário" não entram nos saldos.
- **Juros e multas** só entram nas simulações quando você informa a taxa.

Os cálculos ficam em `src/lib/calc.ts` e são cobertos por testes (`src/lib/calc.test.ts`).

## Estrutura

```
src/
  lib/          cálculos, datas, formatação, validação de backup, CSV
  auth/         tela de login, cofre de usuários e criptografia por conta
  store/        estado global + persistência (criptografada) no LocalStorage
  components/   layout, UI reutilizável, filtros, gráficos
  pages/        Início, Produtividade, Salário, Despesas, Dívidas, Livro-caixa, Planejamento, Relatórios, Configurações
  reports/      geração dos PDFs A4 (jsPDF + AutoTable)
```

## Limitações conhecidas

- Dados locais ao navegador (veja acima); o login é local, não há nuvem nem recuperação de senha.
- Os valores de salário são **estimativas**; confira sempre com o holerite. A regra do adicional noturno e a jornada precisam refletir o seu contrato.
- Os PDFs usam a fonte padrão (Helvetica) e portanto caracteres fora do alfabeto latino não são impressos.
- Alguns navegadores de celular não exibem PDF dentro da página; use "Abrir em nova aba" ou "Baixar PDF".
- A simulação de quitação é aproximada (juros mensais compostos sobre o saldo) e não substitui o cálculo do credor.
- Despesas recorrentes são geradas para uma janela à frente (12 meses para mensais) e completadas automaticamente.
