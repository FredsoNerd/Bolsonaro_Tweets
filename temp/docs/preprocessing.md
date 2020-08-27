# Pré-processamento dos dados

Nessa etapa, iremos realizar o pré-processamento dos dados para apresenta-los em um resultado adequado para a análise exploratória dos dados e em sequência para o desenvolvimento de modelos. É necessário selecionar as informações úteis obtidas dos tweets e organaliza-las em um _dataframe_ do _Pandas_.


```python
import pandas as pd
import pickle as pkl
import tweepy as tw
```

## Preparado os dados obtidos através da API do Twitter

As informações selecionadas para compor o conjunto de dados processado serão as seguintes (e os nomes presente no _dataframe_):

- Nome do ususário que postou o tweet (name).
- Data de criação do tweet (created_at).
- Texto completo do tweet (full_text).
- Hashtags presentes no tweet (hashtags).
- Menções a usuários presentes no tweet (user_mentions).
- Tipo de mídia presente no tweet, se há mídia (media_type).
- Se o tweet é um reply (status_reply).
- Número de retweets que o tweet recebeu (retweet_count).
- Núméro de likes que o tweet recebeu (favorite_count).


```python
#opening API resulted data
with open('..//data//tweets//brute_scrapping.pkl', 'rb') as f:
    tweets_data = pkl.load(f)
```


```python
#create dataframe from tweets data
#going to use the following information from tweets:
#### created_at : date of post
#### full_text: tweet text
#### hashtags: all hashtags in the tweet
#### user_mentions: all mentions in the tweet
#### media_type: type of the media in the tweet, if there is media
#### status_reply: if the tweet is a reply to a status
#### name: username @
#### retweet_count: number of retweets
#### favorite_count: number of retweets
tweets_dict = {}
columns_list = ['created_at', 'full_text', 'hashtags', 'user_mentions', 'media_type', 
                'status_reply','name', 'retweet_count', 'favorite_count'] 
for var in columns_list:
    tweets_dict[var] = []

for user in ['jairbolsonaro', 'bolsonarosp', 'flaviobolsonaro', 'carlosbolsonaro']:
    for tweet in tweets_data[user]:
        for var in columns_list:
            if var == 'hashtags':
                aux = '/'.join([u['text'] for u in tweet._json['entities'][var]])
                if aux == "":
                    tweets_dict[var].append("None")
                else:
                    tweets_dict[var].append(aux)
            elif var == "user_mentions":
                aux = '/'.join([u['screen_name'] for u in tweet._json['entities'][var]])
                if aux== "":
                    tweets_dict[var].append("None")
                else:
                    tweets_dict[var].append(aux)
            elif var == "status_reply":
                tweets_dict[var].append(1 if tweet._json['in_reply_to_status_id'] != None else 0)
            elif var == "name":
                tweets_dict[var].append(user)
            elif var == "media_type":
                if 'media' in list(tweet._json['entities'].keys()):
                    tweets_dict[var].append(tweet._json['extended_entities']['media'][0]['type'])
                else:
                    tweets_dict[var].append(None)
            else:
                tweets_dict[var].append(tweet._json[var])
tweets_df = pd.DataFrame(tweets_dict)
```

Em seguida, iremos criar variáveis extras a partir da variável da data de postagem do tweet, que serão as seguintes:

- Ano de postagem (year).
- Mês de postagem (month).
- Dia de postagem (day).
- Hora de postagem (hour).
- Minuto de postagem (minute).
- Dia da semana de postagem (weekday).

Essas variáveis auxiliarão para o processo de modelagem. Além delas, iremos criar variáveis indicadoras, que serão:

- Se o tweet possui hashtags (has_hashtags).
- Se o tweet possui menções (has_mentions).
- Se o tweet possui alguma mídia (has_midia).


```python
tweets_df['date'] = pd.to_datetime(tweets_df.created_at)
tweets_df['year'] = tweets_df.date.apply(lambda x : x.year)
tweets_df['month'] = tweets_df.date.apply(lambda x : x.month)
tweets_df['day'] = tweets_df.date.apply(lambda x : x.day)
tweets_df['hour'] = tweets_df.date.apply(lambda x : x.hour)
tweets_df['minute'] = tweets_df.date.apply(lambda x : x.minute)
tweets_df['weekday'] = tweets_df.date.apply(lambda x : x.weekday)
tweets_df['has_hashtags'] = tweets_df.hashtags.apply(lambda x : 1 if x != "None" else 0)
tweets_df['has_mentions'] = tweets_df.user_mentions.apply(lambda x : 1 if x != "None" else 0)
tweets_df['has_media'] = tweets_df.media_type.apply(lambda x : 1 if x != "None" else 0)
tweets_df.drop(columns = ['created_at'], inplace = True)
```


```python
tweets_df.to_csv("..\\data\\tweets\\preprocessed_tweets.csv", sep = "~")
```
