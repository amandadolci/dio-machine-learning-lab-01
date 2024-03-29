# Trabalhando com Machine Learning na Prática no Azure ML - Laboratório 1

## Crie um espaço de trabalho do Azure Machine Learning

1. Entre no portal do Azure usando https://portal.azure.com suas credenciais da Microsoft.
2. Selecione **+ Criar um recurso**, pesquise Machine Learning e crie um novo recurso **do Azure Machine Learning com as seguintes configurações:**
   - **Assinatura:** *sua assinatura do Azure*.
   - **Grupo de recursos (Resource group):** *Crie ou selecione um grupo de recursos*.
   - **Nome:** *Insira um nome exclusivo para seu espaço de trabalho*.
   - **Região:** *Selecione a região geográfica mais próxima*.
   - **Conta de armazenamento (Storage account):** *observe a nova conta de armazenamento padrão que será criada para seu espaço de trabalho*.
   - **Cofre de chaves (Key vault):** *Observe o novo cofre de chaves padrão que será criado para seu espaço de trabalho*.
   - **Insights de aplicativo (Application insights):** *observe o novo recurso padrão de insights de aplicativo que será criado para seu espaço de trabalho*.
   - **Registro de contêiner (Container registry):** *Nenhum ( um será criado automaticamente na primeira vez que você implantar um modelo em um contêiner)*.
3. Selecione **Revisar + criar** e selecione **Criar**. Aguarde a criação do seu espaço de trabalho (pode demorar alguns minutos) e, em seguida, vá para o recurso implantado.
4. Selecione **Launch Studio** (ou abra uma nova guia do navegador e navegue até https://ml.azure.com e entre no Azure Machine Learning Studio usando sua conta da Microsoft). Feche todas as mensagens exibidas.
5. No estúdio Azure Machine Learning, você deverá ver seu espaço de trabalho recém-criado. Caso contrário, selecione **Todos os espaços de trabalho** no menu à esquerda e selecione o espaço de trabalho que você acabou de criar.

## Use aprendizado de máquina automatizado para treinar um modelo

1. No Azure Machine Learning Studio, veja a página **Automated ML (em Authoring)**.
2. Crie um novo trabalho de ML automatizado com as seguintes configurações, usando **Next** conforme necessário para avançar pela interface do usuário:

   - **Configurações básicas:**
     - **Nome do trabalho:** mslearn-bike-automl
     - **Novo nome do experimento:** mslearn-bike-rental
     - **Descrição:** Aprendizado de máquina automatizado para previsão de aluguel de bicicletas
     - **Marcadores:** *nenhum*
   - **Tipo de tarefa e dados:**
     - **Selecione o tipo de tarefa:** Regressão
     - **Selecionar conjunto de dados:** crie um novo conjunto de dados com as seguintes configurações:
       - **Tipo de dados:**
         - **Nome:** alugueldebicicletas
         - **Descrição:** dados históricos de aluguel de bicicletas
         - **Tipo:** Tabular
       - **Fonte de dados:**
         - Selecione **Dos arquivos da web**
       - URL da Web:
         - **URL da Web:** https://aka.ms/bike-rentals
         - **Ignorar validação de dados:** *não selecionar*
       - **Configurações:**
         - **Formato de arquivo:** Delimitado
         - **Delimitador:** Vírgula
         - **Codificação:** UTF-8
         - **Cabeçalhos de coluna:** apenas o primeiro arquivo possui cabeçalhos
         - **Pular linhas:** Nenhum
         - **O conjunto de dados contém dados multilinhas:** *não selecione*
       - **Esquema:**
         - Incluir todas as colunas exceto **Caminho (Path)**
         - Revise os tipos detectados automaticamente
         - Selecione **Criar**. Após a criação do conjunto de dados, selecione o conjunto de dados **de alugueldebicicletas** para continuar a enviar o trabalho de ML automatizado.
   - **Configurações de tarefa:**
     - **Tipo de tarefa:** Regressão
     - **Conjunto de dados:** alugueldebicicletas
     - **Coluna de destino:** Rentals (integer)
     - **Configurações adicionais:**
       - **Métrica primária:** raiz do erro quadrático médio normalizado (Normalized root mean squared error)
       - **Explique o melhor modelo:** *Não selecionado*
       - **Usar todos os modelos suportados:** *Desmarcado. Você restringirá o trabalho para tentar apenas alguns algoritmos específicos.*
       - **Modelos permitidos:** *Selecione apenas **RandomForest** e **LightGBM** — normalmente você gostaria de tentar o máximo possível, mas cada modelo adicionado aumenta o tempo necessário para executar o trabalho.*
     - **Limites:** *expanda esta seção*
       - **Máximo de testes (Max trials):** 3
       - **Máximo de testes simultâneos (Max concurrent trials):** 3
       - **Máximo de nós (Max nodes):** 3
       - **Limite de pontuação da métrica (Metric score threshold):** 0,085 *(para que, se um modelo atingir uma pontuação da métrica de erro quadrático médio normalizado de 0,085 ou menos, o trabalho termina.)*
       - **Tempo limite (Timeout):** 15
       - **Tempo limite de iteração (Iteration timeout):** 15
       - **Habilitar rescisão antecipada (Enable early termination):** *selecionado*
     - **Validação e teste:**
       - **Tipo de validação:** divisão de validação de trem (Train-validation split)
       - **Porcentagem de dados de validação:** 10
       - **Conjunto de dados de teste:** Nenhum
   - **Calcular:**
     - **Selecione o tipo de computação:** sem servidor (Serverless)
     - **Tipo de máquina virtual:** CPU
     - **Camada de máquina virtual:** Dedicada
     - **Tamanho da máquina virtual:** `Standard_DS3_V2\*`
     - **Número de instâncias:** 1

3. Envie o trabalho de treinamento. Ele inicia automaticamente.

4. Espere o trabalho terminar. Pode demorar um pouco – agora pode ser um bom momento para uma pausa para o café!

## Avalie o melhor modelo

1. Na guia **Visão geral (Overview)** do trabalho automatizado de aprendizado de máquina, observe o melhor resumo do modelo.
2. Selecione o texto em **Nome do Algoritmo** do melhor modelo para visualizar seus detalhes.
3. Selecione a guia **Métricas** e selecione os gráficos **residuals** e **predicted_true** se eles ainda não estiverem selecionados.
4. Revise os gráficos que mostram o desempenho do modelo. O gráfico de **residuals** mostra os *resíduos* (as diferenças entre os valores previstos e reais) como um histograma. O gráfico **predicted_true** compara os valores previstos com os valores verdadeiros.

## Implantar e testar o modelo

1. Na guia **Modelo** do melhor modelo treinado pelo seu trabalho automatizado de machine learning, selecione **Implantar (Deploy)** e use a opção de **serviço Web** para implantar o modelo com as seguintes configurações:
   - **Nome:** predict-rentals
   - **Descrição:** Prever aluguel de bicicletas
   - **Tipo de computação (Compute type):** Instância de - Contêiner do Azure (Azure Container Instance)
   - **Habilitar autenticação:** *selecionado*
2. Aguarde o início da implantação – isso pode levar alguns segundos. O **status de implantação** do ponto de extremidade **de predict-rentals** será indicado na parte principal da página como *Running*.
3. Aguarde até que o **status da implantação** mude para *Succeeded*. Isso pode levar de 5 a 10 minutos.

## Testar o serviço implantado

1. No estúdio Azure Machine Learning, no menu esquerdo, selecione **Pontos de extremidade (Endpoints)** e abra o ponto de extremidade em tempo real **de predict-rentals**.
2. Na página do ponto de extremidade em tempo real**de predict-rentals, visualize a guia Testar.**
3. No painel **Dados de entrada para testar o ponto de extremidade**, substitua o modelo JSON pelos seguintes dados de entrada:

```json
{
	"Inputs": {
		"data": [
			{
				"day": 1,
				"mnth": 1,
				"year": 2022,
				"season": 2,
				"holiday": 0,
				"weekday": 1,
				"workingday": 1,
				"weathersit": 2,
				"temp": 0.3,
				"atemp": 0.3,
				"hum": 0.3,
				"windspeed": 0.3
			}
		]
	},
	"GlobalParameters": 1.0
}
```

4. Clique no botão **Testar**.
5. Revise os resultados do teste, que incluem um número previsto de aluguéis com base nos recursos de entrada - semelhante a este:

```json
{
	"Results": [444.27799]
}
```

5. O painel de teste pegou os dados de entrada e usou o modelo treinado para retornar o número previsto de aluguéis.

Vamos revisar o que você fez. Você usou um conjunto de dados históricos de aluguel de bicicletas para treinar um modelo. O modelo prevê o número de alugueres de bicicletas esperados num determinado dia, com base em características sazonais e meteorológicas .

## Limpar

- O serviço web que você criou está hospedado em uma *instância de contêiner do Azure*. Se não pretender experimentá-lo ainda mais, deverá eliminar o ponto de extremidade para evitar acumular utilização desnecessária do Azure.
- No estúdio Azure Machine Learning, na guia **Pontos de extremidade**, selecione o ponto de extremidade **de predict-rentals**. Em seguida, selecione **Excluir** e confirme que deseja excluir o ponto de extremidade.
- Excluir sua computação garante que sua assinatura não será cobrada por recursos de computação. No entanto, será cobrada uma pequena quantia pelo armazenamento de dados, desde que o espaço de trabalho do Azure Machine Learning exista na sua assinatura. Se tiver terminado de explorar o Azure Machine Learning, poderá eliminar o espaço de trabalho Azure Machine Learning e os recursos associados.

Para excluir seu espaço de trabalho:

- No portal Azure, na página **Grupos de recursos**, abra o grupo de recursos que especificou ao criar o seu espaço de trabalho Azure Machine Learning.
- Clique em **Excluir grupo de recursos**, digite o nome do grupo de recursos para confirmar que deseja excluí-lo e selecione **Excluir**.
