# Mattermost

## Script de limpeza

A versão free do mattermost (Team Edition) não dá suporte a limpeza automatizada das mensagens\mídias. Para resolver isto, o repo https://github.com/aljazceru/mattermost-retention criou um script de limpeza dos anexos e mensagens antigas.
 
Basicamente você configura os dados do banco e a pasta 'data' do MM, tudo anterior ao período de retenção definido será apagado. O processo demora um pouco mas resolve bem o problema de acúmulo de mensagens e anexos.

Recentemente eu fiz uma PR adicionando suporte ao Mysql, pois a versão da epoca só aceitava Postgres.

