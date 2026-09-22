# MUSE-releases

O que o **app MUSE** baixa sem login: os dados de trânsito do modo offline e o
firmware do painel. O MUSE é um painel de guidão para motocicleta — app Android,
firmware do painel e carcaça impressa.

**Este repositório não tem código e não roda nada.** Ele só guarda o que é
publicado, como anexos de releases de etiqueta fixa. O projeto do MUSE fica num
repositório privado, e o GitHub não entrega anexo de release privada sem login.
Como o app baixa sem login, o que ele baixa precisa morar num repositório
público. É para isso que este existe.

---

## As duas releases

| Etiqueta | O que guarda | Com que frequência |
|---|---|---|
| **`offline-data`** | Os alertas da via e a camada de vias, por estado | Alertas todo dia; vias no dia 1 de cada mês |
| **`firmware`** | O firmware do painel, que o app instala por Bluetooth | A cada versão, publicada à mão |

### `offline-data`

<https://github.com/Memphyz/MUSE-releases/releases/tag/offline-data>

| Arquivo | O que tem |
|---|---|
| `manifest.json` | O índice dos alertas: versão do formato, quando a rodada foi gerada (`generatedAt`, em UTC), as fontes com a data e a licença de cada uma, e o tamanho de cada estado |
| `regions.json.gz` | Os contornos dos 27 estados (IBGE), para saber em que estado cada ponto está |
| `AC.json.gz` … `TO.json.gz` | Os alertas de cada estado: radares, lombadas, pedágios, semáforos, trechos com acidentes |
| `source-*.json.gz` | Cópias das fontes do governo da última rodada boa. O gerador usa quando uma fonte não responde; o app ignora |
| `roads-manifest.json` | O índice da camada de vias, separado do de alertas: ela é muito maior e se atualiza uma vez por mês |
| `roads-XX.ndjson.gz` | As vias do estado, para baixar inteiras |
| `roads-XX.tiles` | As mesmas vias, quadrado por quadrado, para o app ler só as da rota por HTTP Range, sem baixar o estado |

Cada estado de alertas é uma lista de elementos no formato `out center` do
Overpass (a API de consulta do OpenStreetMap). As fontes do governo entram
traduzidas para as tags equivalentes do OSM, e o que não tem tag no OSM, como os
trechos com acidentes, entra com prefixo `muse:`:

```json
{ "format": 2, "uf": "SP", "generatedAt": "…", "elements": [
  { "type": "node", "id": 123, "lat": -23.56, "lon": -46.65, "tags": { "highway": "speed_camera", "maxspeed": "50" } },
  { "type": "node", "id": -3000000000017, "lat": -22.52, "lon": -43.23, "tags": { "muse:accidents": "30", "muse:deaths": "2" } }
] }
```

Os alertas do Brasil inteiro somam poucos megabytes compactados. As vias somam
cerca de **400 MB** (São Paulo sozinho, ~54 MB): é por isso que elas têm índice
próprio, rodada mensal, e no app são opcionais e só se atualizam sozinhas em
rede sem franquia.

### `firmware`

<https://github.com/Memphyz/MUSE-releases/releases/tag/firmware>

| Arquivo | O que tem |
|---|---|
| `firmware-manifest.json` | A versão mais recente, o tamanho, o SHA-256 e as notas de cada versão |
| `muse-fw-X.Y.Z.bin` | O firmware da moto — o que vai instalado no painel |
| `muse-fw-X.Y.Z-bench.bin` | O de bancada, que não dorme sem a chave de ignição |

O app oferece a atualização, mostra as notas da versão e transfere por Bluetooth.
Ele instala **só o build do mesmo tipo** que o painel diz ter: o da moto num
painel de bancada dormiria logo depois de ligar.

Os `.bin` antigos ficam: o app permite voltar para uma versão anterior
instalando de um arquivo.

---

## Como é atualizado

Tudo sai de workflows do GitHub Actions no repositório do MUSE, que escrevem
aqui com um token *fine-grained* restrito a este repositório (*Contents: Read
and write*), guardado nos secrets de lá.

- **Alertas, todo dia às 05:00 (Brasília).** Também pode ser rodado à mão.
- **Vias, no dia 1 de cada mês às 08:00.** Três horas depois da rodada de
  alertas, para as duas nunca publicarem ao mesmo tempo.
- **Firmware, à mão**, a cada versão nova.

Os estados sobem primeiro e o índice por último, cada arquivo trocado no lugar.
Quem baixar no meio da troca ainda lê o índice antigo, que continua batendo com
os arquivos.

Se uma fonte do governo não responder, a rodada usa a cópia da rodada anterior
(`source-*.json.gz`). Se o OpenStreetMap ou o IBGE falharem, a rodada inteira
falha e a publicação anterior continua valendo.

## Como o app usa

Ao abrir, no máximo a cada 12 horas, o app baixa só o `manifest.json`. Se o
`generatedAt` for de uma rodada mais nova que a do aparelho, ele baixa de novo os
estados que o motorista já tinha escolhido. Dentro da área baixada, o app
responde do próprio banco, sem consultar a rede. O dado baixado não expira: sem
rede, o app continua com o que tem.

---

## Cuidados para não quebrar o app

- **Mantenha este repositório público.** Privado, o download do app dá 404.
- **Não apague as releases `offline-data` e `firmware`, nem mude as etiquetas.**
  A etiqueta faz parte do endereço gravado no app. Outras releases aqui não
  atrapalham, porque o app usa essas etiquetas fixas e nunca a "latest".
  Se uma release for apagada por engano, o workflow correspondente a recria na
  rodada seguinte — mas a `offline-data` também guarda o `regions.json.gz`, do
  qual a rodada de vias depende para não ir ao IBGE. Rode **alertas antes de
  vias** para se recuperar.
- **Não renomeie este repositório** sem antes acertar o `PublicReleases.kt` no
  MUSE e publicar um app novo: o endereço fica gravado no app instalado.
- **Não arquive o repositório.** Repositório arquivado não aceita anexo novo, e
  tudo para de atualizar.
- **Não coloque os pacotes no git.** Eles vivem só como anexos das releases, que
  os workflows trocam.

---

## Fontes e licenças

| Fonte | O que vira | Licença |
|---|---|---|
| OpenStreetMap, extrato do Brasil da Geofabrik | Tudo o que os filtros de alerta do app encontram (radares, lombadas, semáforos, PARE, pedágios, cruzamentos de trem…) e a camada de vias, com o limite de velocidade e o piso | ODbL |
| ANTT: radares e praças de pedágio das rodovias federais concedidas | Radares com o limite de velocidade, e praças de pedágio | CC-BY |
| PRF: boletins de acidentes (últimos 3 anos) | Trechos de 1 km com 10 ou mais acidentes | Não conferida |
| IBGE: malhas territoriais | Contornos dos estados | — |

Os dados do OpenStreetMap são **© colaboradores do OpenStreetMap**, sob a
[Open Database License (ODbL)](https://opendatacommons.org/licenses/odbl/).
Quem reusar precisa dar esse crédito, e um banco derivado distribuído
publicamente segue a mesma licença. O `manifest.json` de cada rodada lista as
fontes que entraram e a data de cada uma.
