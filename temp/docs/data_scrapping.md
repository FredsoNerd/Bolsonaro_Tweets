# Raspagem e preparação dos dados

## Dados do Twitter

A etapa inicial do projeto é a obtenção dos dados do Twitter, faremos o scraping (raspagem) dos dados, para isso utilizaremos de duas fontes, um conjunto de dados presente no Kraggle e um conjunto de dados obtido através da API do Twitter.


```python
import pandas as pd
import tweepy as tw
import pickle as pkl
import getpass
import time
```

### Dados do Kraggle
O conjunto de dados do Kraggle _Jair Bolsonaro Twitter Data_ apresenta uma coleção de 3 mil tweets do Bolsonaro.


```python
kraggle_data = pd.read_csv("..//data//tweets//bolsonaro_tweets.csv")
kraggle_data.date = pd.to_datetime(kraggle_data.date)
kraggle_data.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>text</th>
      <th>likes</th>
      <th>retweets</th>
      <th>link</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2020-08-01</td>
      <td>- Ministro @tarcisiogdf e os 160 anos da Infra...</td>
      <td>880</td>
      <td>163</td>
      <td>https://twitter.com/jairbolsonaro/status/12895...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2020-08-01</td>
      <td>- Coletiva com o prefeito de Bagé/RS, Divaldo ...</td>
      <td>336</td>
      <td>51</td>
      <td>https://twitter.com/jairbolsonaro/status/12895...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2020-08-01</td>
      <td>@rsallesmma 👍🏻</td>
      <td>442</td>
      <td>39</td>
      <td>https://twitter.com/jairbolsonaro/status/12893...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2020-08-01</td>
      <td>@Marceloscf2 @tarcisiogdf Seguimos! 🤝</td>
      <td>164</td>
      <td>22</td>
      <td>https://twitter.com/jairbolsonaro/status/12893...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2020-08-01</td>
      <td>@ItamaratyGovBr @USAID 👍🏻</td>
      <td>223</td>
      <td>24</td>
      <td>https://twitter.com/jairbolsonaro/status/12893...</td>
    </tr>
  </tbody>
</table>
</div>




```python
print("O período de " +  str(len(kraggle_data))+" tweets é de " +  str(kraggle_data.date.min())[0:10] +" até " +  str(kraggle_data.date.max())[0:10] + ".")
```

    O período de 9351 tweets é de 2010-04-01 até 2020-08-01.
    

### Twitter API
Nós também podemos obter dados sobre Jair Bolsonaro, Flávio Bolsonaro, Carlos Bolsonaro e Eduardo Bolsonaro da API do Twitter usando Tweepy. No entanto, o acesso é permitido apenas para os últimos 3200 tweets, dessa forma, não é o histórico completo de tweets.

<details>
<summary>Autenticação</summary>

```python
consumer_key = getpass.getpass()
```

    ········
    


```python
consumer_secret = getpass.getpass()
```

    ········
    


```python
access_token = getpass.getpass()
```

    ········
    


```python
access_token_secret = getpass.getpass()
```

    ········
    


```python
auth = tw.OAuthHandler(consumer_key, consumer_secret)
auth.set_access_token(access_token, access_token_secret)
api = tw.API(auth, wait_on_rate_limit=True)
```

</details>


```python
#requesting data from twitter
twitter_data = {}
for user in ['jairbolsonaro', 'bolsonarosp', 'flaviobolsonaro', 'carlosbolsonaro']:
    twitter_data[user] = [tweet for tweet in tw.Cursor(api.user_timeline,id=user, tweet_mode ="extended").items()]
```


```python
#saving original data
with open("..//data//tweets//brute_scrapping,pkl", "wb+") as f:
    pkl.dump(twitter_data, f)
```


```python
#script to create a dataframe from the data
date = []
text = []
user_name = []
place = []
retweets = []
favorites = []
for user in ['jairbolsonaro', 'bolsonarosp', 'flaviobolsonaro', 'carlosbolsonaro']:
    for tweet in twitter_data[user]:
        date.append(tweet._json['created_at'])
        text.append(tweet._json['full_text'])
        user_name.append(user)
        place.append(tweet._json['place'])
        retweets.append(tweet._json['retweet_count'])
        favorites.append(tweet._json['favorite_count'])
API_data = pd.DataFrame({'name': user_name, 'text': text, 'date': pd.to_datetime(date), 
                         'place': place, 'retweets': retweets, 'favorites':favorites})
API_data.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>text</th>
      <th>date</th>
      <th>place</th>
      <th>retweets</th>
      <th>favorites</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>jairbolsonaro</td>
      <td>-Edifício Joelma/SP, 1974.\n\n-Sgt CASSANIGA s...</td>
      <td>2020-07-27 20:51:13+00:00</td>
      <td>None</td>
      <td>3154</td>
      <td>16202</td>
    </tr>
    <tr>
      <th>1</th>
      <td>jairbolsonaro</td>
      <td>- Água para quem tem sede.\n- Liberdade para u...</td>
      <td>2020-07-27 11:10:36+00:00</td>
      <td>None</td>
      <td>8101</td>
      <td>37357</td>
    </tr>
    <tr>
      <th>2</th>
      <td>jairbolsonaro</td>
      <td>@tarcisiogdf @MInfraestrutura 🤝🇧🇷, Ministro!</td>
      <td>2020-07-26 20:18:19+00:00</td>
      <td>None</td>
      <td>1074</td>
      <td>16840</td>
    </tr>
    <tr>
      <th>3</th>
      <td>jairbolsonaro</td>
      <td>2- @MinEconomia @MinCidadania @onyxlorenzoni @...</td>
      <td>2020-07-26 15:40:39+00:00</td>
      <td>None</td>
      <td>1337</td>
      <td>6383</td>
    </tr>
    <tr>
      <th>4</th>
      <td>jairbolsonaro</td>
      <td>1- Acompanhe as redes sociais! @secomvc @fabio...</td>
      <td>2020-07-26 15:39:47+00:00</td>
      <td>None</td>
      <td>3287</td>
      <td>14836</td>
    </tr>
  </tbody>
</table>
</div>




```python
API_data.to_csv('..API_data.csv', sep = "~")
```

## Dados econômicos

Nós vamos coletar os dados do BACEN (Banco Central do Brasil) usando sua API. Nós vamos utilizar as séries temporais de alguns importantes indicadores econômicos como GDP, IPCA, NPCC. O código presente neste notebook é baseado no vídeo de __Código Quant - Finanças Quantitativas__: [COMO ACESSAR A BASE DE DADOS DO BANCO CENTRAL DO BRASIL COM PYTHON | Python para Investimentos #10](https://www.youtube.com/watch?v=7rFsu48oBn8).


```python
def query_bc(code):
    url = 'http://api.bcb.gov.br/dados/serie/bcdata.sgs.{}/dados?formato=json'.format(code)
    df = pd.read_json(url)
    df['data'] = pd.to_datetime(df['data'], dayfirst=True)
    df.set_index('data', inplace=True)
    return df
```

As séries temporais escolhidas para compor o nosso conjunto de dados estão listadas abaixo, tabmém está listado sua unidade e o instituto que publica o indicador:

- IPCA : Índice nacional de preços ao consumidor-amplo, Var. % mensal, (IBGE). Este índice mede a variação de preços de mercado para o consumidor final e é utilizado para medir a inflação.
- INPC: Índice nacional de preços ao consumidor, Var. % mensal, (IBGE). Este índice mede a variação de preços da cesta de consumo da população assalariada com mais baixo rendimento.
- IGP-M : Índice geral de preços do mercado, Var. % mensal, (FGV). Este índice mede a variação de preços através de outros índices, do IPCA, INCC, e IPC.
- Meta SELIC : Taxa básica de juros definida pelo Copom, % a.a., (Copom).
- CDI : Taxa de juros, % a.d., (Cetip). Taxa de juros para empréstimo entre bancos.
- PIB : Produto Interno Bruto mensal, R\$ (milhões), (BCB-Depec). 
- Dólar: Taxa de câmbio - Dólar americano (venda), u.m.c/US\$, (Sisbacen PTAX800).
- Dívida pública: Dívida líquida do Setor Público (%PIB) - Total, %, (BSB - DSTAT).
- Reserva Internacional: Reserva internacional total diária, US\$ (milhões), (BCB - DSTAT). 
- Índice de emprego formal, (Mtb).
- PNADC: Taxa de desocupação, %, (IBGE). 
- Índice de confiança do consumidor, (Fecomercio).


```python
ipca = query_bc(433) 
ipca.to_csv("..\\data\\economic_index\\ipca.csv", sep = ";")
time.sleep(5)

igpm = query_bc(189) 
igpm.to_csv("..\\data\\economic_index\\igpm.csv", sep = ";")
time.sleep(5)

inpc = query_bc(188)
inpc.to_csv("..\\data\\economic_index\\inpc.csv", sep = ";")
time.sleep(5)

selic_meta = query_bc(432)
selic_meta.to_csv("..\\data\\economic_index\\selic_meta.csv", sep = ";")
time.sleep(5)

international_reserve = query_bc(13621)
international_reserve.to_csv("..\\data\\economic_index\\international_reserve.csv", sep = ";")
time.sleep(5)

pnad = query_bc(24369)
pnad.to_csv("..\\data\\economic_index\\pnad.csv", sep = ";")
time.sleep(5)

cdi = query_bc(12)
cdi.to_csv("..\\data\\economic_index\\cdi.csv", sep = ";")
time.sleep(5)

gdp = query_bc(4385)
gdp.to_csv("..\\data\\economic_index\\gdp.csv", sep = ";")
time.sleep(5)

dollar = query_bc(1)
dollar.to_csv("..\\data\\economic_index\\dollar.csv", sep = ";")
time.sleep(5)

employment = query_bc(25239)
employment.to_csv("..\\data\\economic_index\\employment.csv", sep = ";")
time.sleep(5)

gov_debt = query_bc(4503)
gov_debt.to_csv("..\\data\\economic_index\\gov_debt.csv", sep = ";")
time.sleep(5)

consumer_confidence = query_bc(4393)
consumer_confidence.to_csv("..\\data\\economic_index\\consumer_confidence.csv", sep = ";")
time.sleep(5)
```

Vamos gerar um _dataframe_ final incluindo todas as séries temporais econômicas, note que existirão células vazias pois os indicadores econômicos apresentam frequências diferentes, podem ser diários, mensais, trimestrais ou anuais, e também os indicadores começaram a ser utilizados em momentos distintos na história.


```python
economic_time_series = pd.merge(ipca, igpm, left_index = True, right_index= True, how= 'outer')
economic_time_series = economic_time_series.merge(inpc, left_index = True, right_index= True, how= 'outer')
economic_time_series = economic_time_series.merge(selic_meta, left_index = True, right_index= True, how= 'outer')
economic_time_series = economic_time_series.merge(international_reserve, left_index = True, right_index= True, how= 'outer')
economic_time_series = economic_time_series.merge(pnad,  left_index = True, right_index= True, how= 'outer')
economic_time_series = economic_time_series.merge(cdi, left_index = True, right_index= True, how= 'outer')
economic_time_series = economic_time_series.merge(gdp, left_index = True, right_index= True, how= 'outer')
economic_time_series = economic_time_series.merge(dollar, left_index = True, right_index= True, how= 'outer')
economic_time_series = economic_time_series.merge(employment, left_index = True, right_index= True, how= 'outer')
economic_time_series = economic_time_series.merge(gov_debt, left_index = True, right_index= True, how= 'outer')
economic_time_series = economic_time_series.merge(consumer_confidence, left_index = True, right_index= True, how= 'outer')

economic_time_series.columns = ["ipca", "igpm", "inpc", "selic_meta", "international_reserve",
                                      "pnad", "cdi", "gdp", "dollar", "employment", "gov_debt", "consumer_confidence"]
economic_time_series.to_csv("..\\data\\economic_index\\economic_time_series.csv", sep = ";")
```
