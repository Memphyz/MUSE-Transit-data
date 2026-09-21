# MUSE-Transit-data

Dados de trânsito para o **modo offline do app MUSE**, um painel de guidão para
motocicleta: radares, lombadas, pedágios, semáforos, trechos com acidentes e os
outros alertas da via, separados por estado do Brasil.

**Este repositório não tem código e não roda nada.** Ele só guarda os pacotes de
dados como anexos de uma release pública. O projeto do MUSE (app, firmware do
painel e carcaça) fica num repositório privado, e o GitHub não entrega anexo de
release privada sem login. Como o app baixa os dados sem login, eles precisam
morar num repositório público. É para isso que este existe.

---

## Onde estão os dados

Na release de etiqueta fixa **`dados-offline`**:
<https://github.com/Memphyz/MUSE-Transit-data/releases/tag/dados-offline>

O app baixa de `https://github.com/Memphyz/MUSE-Transit-data/releases/download/dados-offline/`.
Esse endereço está gravado no app instalado.

| Arquivo | O que tem |
|---|---|
| `manifest.json` | O índice: versão do formato, quando a rodada foi gerada (`generatedAt`, em UTC), as fontes com a data e a licença de cada uma, e o tamanho de cada estado |
| `regioes.json.gz` | Os contornos dos 27 estados (IBGE), usados para saber em que estado cada ponto está |
| `AC.json.gz` … `TO.json.gz` | Os alertas de cada estado |
| `fonte-*.json.gz` | Cópias das fontes do governo da última rodada boa. O gerador usa quando uma fonte não responde; o app ignora |

Cada estado é uma lista de elementos no formato `out center` do Overpass (a API
de consulta do OpenStreetMap). As fontes do governo entram traduzidas para as
tags equivalentes do OSM, e o que não tem tag no OSM, como os trechos com
acidentes, entra com prefixo `muse:`:

```json
{ "format": 1, "uf": "SP", "generatedAt": "…", "elements": [
  { "type": "node", "id": 123, "lat": -23.56, "lon": -46.65, "tags": { "highway": "speed_camera", "maxspeed": "50" } },
  { "type": "node", "id": -3000000000017, "lat": -22.52, "lon": -43.23, "tags": { "muse:acidentes": "30", "muse:mortos": "2" } }
] }
```

O Brasil inteiro soma poucos megabytes compactados.

---

## Como é atualizado

- **Toda segunda-feira às 03:00 (Brasília)**, um workflow do GitHub Actions no
  repositório do MUSE gera os pacotes a partir das fontes e troca os anexos desta
  release. Ele também pode ser rodado à mão.
- O workflow escreve aqui com um token *fine-grained* restrito a este
  repositório (*Contents: Read and write*), guardado nos secrets do MUSE.
- Os estados sobem primeiro e o `manifest.json` por último, cada arquivo trocado
  no lugar. Quem baixar no meio da troca ainda lê o índice antigo, que continua
  batendo com os arquivos.
- Se uma fonte do governo não responder, a rodada usa a cópia da rodada anterior
  (`fonte-*.json.gz`). Se o OpenStreetMap ou o IBGE falharem, a rodada inteira
  falha e a publicação anterior continua valendo.

## Como o app usa

Ao abrir, no máximo a cada 12 horas, o app baixa só o `manifest.json`. Se o
`generatedAt` for de uma rodada mais nova que a do aparelho, ele baixa de novo os
estados que o usuário já tinha escolhido. Dentro da área baixada, o app responde
do próprio banco, sem consultar a rede. O dado baixado não expira: sem rede, o
app continua com o que tem.

---

## Cuidados para não quebrar o app

- **Mantenha este repositório público.** Privado, o download do app dá 404.
- **Não apague nem renomeie a release `dados-offline`.** A etiqueta faz parte do
  endereço gravado no app. Outras releases aqui não atrapalham, porque o app usa
  essa etiqueta fixa e nunca a "latest".
- **Não arquive o repositório.** Repositório arquivado não aceita anexo novo, e
  os dados param de atualizar.
- **Não coloque os pacotes no git.** Eles vivem só como anexos da release, que o
  workflow troca toda semana.

---

## Fontes e licenças

| Fonte | O que vira | Licença |
|---|---|---|
| OpenStreetMap, extrato do Brasil da Geofabrik | Tudo o que os filtros de alerta do app encontram: radares, lombadas, semáforos, PARE, pedágios, cruzamentos de trem… | ODbL |
| ANTT: radares e praças de pedágio das rodovias federais concedidas | Radares com o limite de velocidade, e praças de pedágio | CC-BY |
| PRF: boletins de acidentes (últimos 3 anos) | Trechos de 1 km com 10 ou mais acidentes | Não conferida |
| IBGE: malhas territoriais | Contornos dos estados | — |

Os dados do OpenStreetMap são **© colaboradores do OpenStreetMap**, sob a
[Open Database License (ODbL)](https://opendatacommons.org/licenses/odbl/).
Quem reusar precisa dar esse crédito, e um banco derivado distribuído
publicamente segue a mesma licença. O `manifest.json` de cada rodada lista as
fontes que entraram e a data de cada uma.
