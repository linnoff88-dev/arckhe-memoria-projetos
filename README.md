ï»¿# memoria-projetos ê¢â¬â ache qualquer projeto e pergunte o que jêÂ¡ foi decidido

<div align="center">
  <img src="capa/logo.svg" alt="Arckhé AI" height="48" />
  <p><strong>Arckhé AI</strong></p>
</div>


[![MemêÂ³ria de Projetos](guia/assets/banner.jpg)](https://linnoff88-dev.github.io/arckhe-/guia/)

**ê°Å¸â¡Â§ê°Å¸â¡Â· [PortuguêÂªs](README.md) êÂ· ê°Å¸â¡Âºê°Å¸â¡Â¸ [English](README.en.md) êÂ· ê°Å¸â¡Âªê°Å¸â¡Â¸ [EspaêÂ±ol](README.es.md)**

## O que êÂ©

O memoria-projetos êÂ© um kit para quem tem muitos projetos e trabalha com agentes de IA, como o Claude Code e o Codex. Ele resolve dois problemas: o agente nêÂ£o sabe onde estêÂ¡ cada projeto e esquece o que jêÂ¡ foi decidido. O comando `projeto` procura pelo que o projeto faz, nas suas pastas e no GitHub. O comando `memoria` responde perguntas como "pode fazer X?" lendo suas liêÂ§êÂµes e anotaêÂ§êÂµes, com data e fonte. Para usar, vocêÂª precisa de um computador com Linux, Python, o Ollama e, para a memêÂ³ria, a assinatura do Claude (sem chave de API). O êÂ­ndice fica todo na sua mêÂ¡quina; a memêÂ³ria manda o texto das anotaêÂ§êÂµes ao Claude para extrair os fatos. As duas metades sêÂ£o independentes: dêÂ¡ para instalar sêÂ³ uma.

## ê°Å¸ââ Guia de uso

Guia completo (landing + passo a passo): **https://linnoff88-dev.github.io/arckhe-/guia/**

## As duas metades

| Metade | Pasta | Comando | Precisa de |
|---|---|---|---|
| êÂndice de projetos | `indice/` | `projeto` | Python 3, Ollama com `bge-m3`, `gh` (opcional) |
| MemêÂ³ria que responde | `memoria/` | `memoria` | Python 3, servidor Hindsight, Claude Code CLI logado, Ollama com `bge-m3` |

## PrêÂ©-requisitos

Instale e confira antes (o `instalar.sh` sêÂ³ confere, nêÂ£o instala nada do sistema):

1. **Python 3.11 ou mais novo** (`python3 --version`). Os comandos do kit usam sêÂ³ a biblioteca padrêÂ£o.
2. **Ollama** (https://ollama.com) rodando em `127.0.0.1:11434`, com o modelo de vetores multilêÂ­ngue:
   ```bash
   ollama pull bge-m3
   ```
3. **SêÂ³ para o êÂ­ndice ê¢â¬â GitHub CLI** (`gh`), logado em cada conta que vocêÂª quer indexar:
   ```bash
   gh auth login            # repita para cada conta; o kit usa "gh auth token -u <conta>"
   gh auth status
   ```
   Sem `gh` (ou com `"contas": []`), o êÂ­ndice fica sêÂ³ com as pastas locais.
4. **SêÂ³ para a memêÂ³ria ê¢â¬â Claude Code CLI** instalado e logado na sua assinatura (`claude` no PATH; o kit pêÂµe `~/.npm-global/bin` e `~/.local/bin` na frente). O servidor usa o provedor `claude-code` do Hindsight, que a documentaêÂ§êÂ£o dele indica para **uso pessoal/local**.
5. **SêÂ³ para a memêÂ³ria ê¢â¬â internet uma vez** na primeira subida do servidor: o PostgreSQL embutido (pg0) e o modelo de reordenaêÂ§êÂ£o sêÂ£o baixados nessa hora. Depois, tudo roda local.
6. **systemd de usuêÂ¡rio** (Linux) para os horêÂ¡rios automêÂ¡ticos. Para os timers rodarem mesmo sem vocêÂª logado:
   ```bash
   loginctl enable-linger "$USER"
   ```
   Sem systemd, use `--sem-systemd` e rode os comandos êÂ  mêÂ£o (ou pelo cron).

## InstalaêÂ§êÂ£o

```bash
git clone https://github.com/linnoff88-dev/memoria-projetos.git
cd memoria-projetos
./instalar.sh --dry-run                 # mostra tudo o que faria, sem gravar nada
./instalar.sh --instalar-hindsight      # instala as duas metades + o servidor Hindsight
```

OpêÂ§êÂµes do `instalar.sh` (pode rodar quantas vezes quiser: sêÂ³ grava o que mudou):

| OpêÂ§êÂ£o | O que faz |
|---|---|
| `--so-indice` / `--so-memoria` | instala sêÂ³ uma metade |
| `--sem-systemd` | nêÂ£o cria serviêÂ§os nem timers |
| `--dry-run` | sêÂ³ mostra o que faria |
| `--instalar-hindsight` | cria `memoria/.venv` e instala o Hindsight (ê¢â°Ë2 GB) |
| `--forcar` | substitui atalho ou unidade que jêÂ¡ existe e nêÂ£o êÂ© deste kit |

O que ele faz:

- copia `config.exemplo.json` ê¢â â `config.json`, `sinonimos.exemplo.json` ê¢â â `sinonimos.json`, `.env.exemplo` ê¢â â `.env` e `fontes.exemplo.json` ê¢â â `fontes.json` **sêÂ³ se ainda nêÂ£o existirem** (nunca apaga config sua);
- cria os atalhos `~/.local/bin/projeto` e `~/.local/bin/memoria` apontando para esta pasta;
- escreve as unidades do systemd de usuêÂ¡rio com o caminho real desta pasta, com teto de memêÂ³ria (`MemoryMax`) e de tempo (`TimeoutStartSec`), `PATH` com `~/.npm-global/bin` e `~/.local/bin` na frente e timers `Persistent=true` (se a mêÂ¡quina estava desligada no horêÂ¡rio, roda quando ligar):

| Unidade | Quando | O que roda | Teto |
|---|---|---|---|
| `memoria-indice.timer` | todo dia 05:40 | `projeto --atualizar` | 1 GB êÂ· 20 min |
| `memoria-hindsight.service` | sempre ligado | servidor Hindsight em `127.0.0.1:8891` | 8 GB |
| `memoria-carga.timer` | todo dia 06:10 | `carga.py --max 25` | 512 MB êÂ· 60 min |
| `memoria-semana.timer` | segunda 08:00 | `semana.py --dias 7` | 512 MB êÂ· 30 min |

### O Hindsight, se preferir instalar êÂ  mêÂ£o

O `--instalar-hindsight` faz exatamente isto (versêÂ£o 1.0.0, a que foi testada; a 1.2.0 jêÂ¡ existe no PyPI):

```bash
python3 -m venv memoria/.venv
memoria/.venv/bin/pip install torch --index-url https://download.pytorch.org/whl/cpu   # torch sêÂ³ CPU (bem menor)
memoria/.venv/bin/pip install hindsight-api==1.0.0
```

Se o `hindsight-api` jêÂ¡ estiver instalado em outro lugar: `HINDSIGHT_BIN=/caminho/hindsight-api ./instalar.sh`.
TambêÂ©m funciona a partir do cêÂ³digo-fonte: `pip install -e "<pasta-do-clone>/hindsight-api-slim[all]"` (clone de https://github.com/vectorize-io/hindsight).
Os dados do banco ficam em `~/.pg0` (PostgreSQL embutido).

## ConfiguraêÂ§êÂ£o

### êÂndice ê¢â¬â `indice/config.json`

| Campo | Para que serve |
|---|---|
| `raiz` | pasta onde ficam seus projetos (ex.: `~/projetos`); cada subpasta com git ou README vira um projeto |
| `contas` | contas do GitHub a indexar (`[]` = sêÂ³ local). A ordem conta: em pasta com vêÂ¡rios remotos, a 1êÂª conta ganha |
| `subpastas_com_readme` | pastas (relativas êÂ  raiz) cujas subpastas com README tambêÂ©m contam como projeto (ex.: uma pasta de ferramentas) |
| `pular` / `pular_subpastas` | nomes de pasta que nunca entram |
| `saida_md` / `saida_jsonl` | onde gravar o êÂ­ndice (padrêÂ£o: dentro de `indice/`) |
| `ollama_url` / `ollama_modelo` | onde estêÂ£o os vetores (padrêÂ£o `bge-m3` local) |
| `peso_palavra` | quanto a busca por palavra soma êÂ  busca por sentido (padrêÂ£o 0,35) |

`sinonimos.json`: abreviaêÂ§êÂµes que valem como a palavra inteira (`yt` ê¢â â `youtube`, `ops` ê¢â â `operacao`ê¢â¬Â¦). Acrescente as suas.

### MemêÂ³ria ê¢â¬â `memoria/.env` e `memoria/fontes.json`

No `.env`: `HINDSIGHT_URL`, `HINDSIGHT_BANCO` (um banco por projeto ou pessoa) e a configuraêÂ§êÂ£o do servidor (LLM `claude-code` com `claude-haiku-5-5`, vetores `bge-m3` no Ollama, reordenaêÂ§êÂ£o local). A linha da chave do Ollama êÂ© um texto qualquer: o Ollama nêÂ£o usa chave.

No `fontes.json`, a lista do que vai para a memêÂ³ria. TrêÂªs tipos:

| Tipo | LêÂª | Vira |
|---|---|---|
| `linhas` | itens `- ` de uma lista markdown (ex.: `LESSONS.md`) | as linhas novas do dia, em blocos de atêÂ© `bloco` linhas (padrêÂ£o 15) |
| `tabela` | linhas de dados de tabelas markdown (ex.: `FALHAS.md`), sem cabeêÂ§alho | idem |
| `pasta-md` | cada arquivo `.md` de uma pasta (ex.: a memêÂ³ria do agente) | um documento por arquivo; se o arquivo mudar, substitui o anterior |

Cada fonte tem `nome` (nêÂ£o troque depois da 1êÂª carga), `caminho`, `contexto` (uma frase dizendo o que êÂ© aquilo) e `tag`. Em `pasta-md`, `ignorar` lista arquivos que ficam de fora (ex.: o êÂ­ndice `MEMORY.md`).

**Custo da carga.** A carga sêÂ³ envia o que êÂ© novo ou mudou; conteêÂºdo idêÂªntico nem sai da mêÂ¡quina. Mas cada documento enviado êÂ© uma chamada ao LLM pela sua assinatura. Medido no piloto: **~5,6 mil tokens de entrada do Haiku por documento; 100 documentos ê¢â°Ë 8 min ê¢â°Ë 566 mil tokens**. Por isso o teto `--max` (padrêÂ£o 25 por execuêÂ§êÂ£o; o que sobrar vai no dia seguinte). A 1êÂª execuêÂ§êÂ£o êÂ© a carga inicial: veja antes com `--simular` e, se quiser fazer tudo de uma vez, use `--max 200`.
AtenêÂ§êÂ£o: editar uma linha antiga (em `linhas` ou `tabela`) custa duas chamadas: o bloco antigo êÂ© reenviado sem ela e a versêÂ£o nova entra no bloco do dia. Acrescentar linhas novas nêÂ£o mexe nos blocos antigos.

## Uso

```bash
# êÂ­ndice
projeto --atualizar                      # refaz o êÂ­ndice e os vetores (ê¢â°Ë1ê¢â¬â2 min para mil projetos)
projeto "separa operaêÂ§êÂ£o do desenvolvimento"
projeto "publica vêÂ­deo no youtube" -n 5  # * na nota = todas as palavras casaram

# memêÂ³ria
python3 memoria/carga.py --simular       # o que seria gravado (nêÂ£o envia nada)
python3 memoria/carga.py --max 200       # 1êÂª carga
memoria "posso rodar processo longo sem teto de memêÂ³ria?"   # resposta pronta (30ê¢â¬â75 s)
memoria -r "systemd PATH"                # sêÂ³ os trechos (7ê¢â¬â17 s, sem LLM na resposta)
memoria --nota util "achou a regra certa"                   # avalia a êÂºltima resposta
python3 memoria/semana.py                # relatêÂ³rio da semana ê¢â â memoria/semana-AAAA-MM-DD.md
python3 memoria/avalia.py                # grep êâ recall êâ reflect nas suas perguntas (gasta cota)
```

Se o Ollama estiver fora, o `projeto` cai sêÂ³ na busca por palavra e avisa. Se o servidor da memêÂ³ria estiver fora, o `memoria` avisa e sai com cêÂ³digo 2: use o grep de sempre.

`avalia.py` usa `perguntas.jsonl` (copie de `perguntas.exemplo.jsonl` e escreva perguntas reais suas), uma por linha: `{"pergunta": "...", "palavra_grep": "palavra que vocêÂª usaria no grep", "gabarito": ["regex que a resposta certa precisa ter"]}`.

## O processo, do inêÂ­cio ao fim

ê°Å¸Â§Â  sessêÂ£o (vocêÂª, com ou sem o agente) êÂ· ê¢Å¡â¢ê¯Â¸Â script/timer êÂ· ê°Å¸Â¤â agente

1. ê°Å¸Â§Â  Instalar os prêÂ©-requisitos (Ollama + `ollama pull bge-m3`, `gh auth login`, Claude Code logado, `loginctl enable-linger`).
2. ê°Å¸Â§Â  `./instalar.sh --dry-run` e depois `./instalar.sh --instalar-hindsight`.
3. ê°Å¸Â§Â  Ajustar `indice/config.json` (raiz e contas), `memoria/fontes.json` (suas fontes) e `memoria/.env` (nome do banco).
4. ê°Å¸Â§Â  Primeiro êÂ­ndice: `projeto --atualizar`; conferir com `projeto "algo que vocêÂª sabe onde estêÂ¡"`.
5. ê°Å¸Â§Â  Primeira carga da memêÂ³ria: `python3 memoria/carga.py --simular`, depois `python3 memoria/carga.py --max 200`.
6. ê°Å¸Â§Â  Colar a regra do agente (abaixo) no `CLAUDE.md` / `AGENTS.md` dos seus projetos.
7. ê¢Å¡â¢ê¯Â¸Â Todo dia 05:40: `memoria-indice.timer` refaz o êÂ­ndice e calcula vetores sêÂ³ do que mudou.
8. ê¢Å¡â¢ê¯Â¸Â Todo dia 06:10: `memoria-carga.timer` grava na memêÂ³ria sêÂ³ o que êÂ© novo (no mêÂ¡ximo 25 documentos).
9. ê°Å¸Â¤â Na sessêÂ£o, pergunta "qual projeto faz X / onde estêÂ¡ Y" ê¢â â `projeto "X"` antes de varrer pastas.
10. ê°Å¸Â¤â Na sessêÂ£o, pergunta de regra ou decisêÂ£o ("pode X?", "o que decidimos sobre Y?") ê¢â â `memoria "ê¢â¬Â¦"` antes do grep; depois `memoria --nota util|inutil "motivo"`.
11. ê¢Å¡â¢ê¯Â¸Â Toda segunda 08:00: `memoria-semana.timer` gera `memoria/semana-AAAA-MM-DD.md` (uso, adoêÂ§êÂ£o pelo agente, custo, RAM).
12. ê°Å¸Â§Â  Ler o relatêÂ³rio: ficou êÂºtil, muda a regra do agente ou desliga. De vez em quando, `python3 memoria/avalia.py` para medir a qualidade.

Desligar tudo: `systemctl --user disable --now memoria-hindsight.service memoria-carga.timer memoria-semana.timer memoria-indice.timer`.

## Fazendo o agente usar

O agente sêÂ³ usa o que estêÂ¡ na regra dele. Cole isto no `CLAUDE.md` (Claude Code) ou no `AGENTS.md` (Codex) dos seus projetos, ou no arquivo global de instruêÂ§êÂµes:

```markdown
## Achar projeto e consultar a memêÂ³ria

- Pergunta do tipo "qual projeto faz X" ou "onde estêÂ¡ Y": rode `projeto "X"` ANTES de varrer pastas
  ou o histêÂ³rico. NêÂ£o leia o projetos.md inteiro; use o comando ou `grep -i`.
- Pergunta de regra, decisêÂ£o ou erro jêÂ¡ visto ("pode X?", "o que decidimos sobre Y?",
  "que erro jêÂ¡ deu com Z?"): rode `memoria "pergunta"` (resposta pronta) ou `memoria -r "palavras"`
  (sêÂ³ trechos, mais rêÂ¡pido) ANTES do grep nas liêÂ§êÂµes e falhas.
- Depois de usar a resposta, avalie: `memoria --nota util "motivo"` ou `memoria --nota inutil "motivo"`.
- Se `memoria` sair com cêÂ³digo 2 (servidor fora), siga com o grep de sempre e avise.
```

NêÂ£o coloque a resposta da memêÂ³ria no inêÂ­cio de toda sessêÂ£o: o agente pergunta quando precisa. Assim o comeêÂ§o da sessêÂ£o fica sempre igual (o cache funciona) e a cota sêÂ³ êÂ© gasta quando hêÂ¡ pergunta.

## CrêÂ©ditos

- A memêÂ³ria usa o **Hindsight**, da vectorize-io, licenêÂ§a MIT: https://github.com/vectorize-io/hindsight
- Espelho e guia em portuguêÂªs: https://github.com/linnoff88-dev/hindsight êÂ· https://linnoff88-dev.github.io/arckhe-/guia/
- Vetores: modelo `bge-m3` (BAAI) servido pelo Ollama.

## LicenêÂ§a

MIT ê¢â¬â veja [LICENSE](LICENSE). êÂ© 2026 ARCKHÉ.

---

<div align="center">
  <img src="capa/logo.svg" alt="Arckhé AI" height="32" />
  <p><strong>Arckhé AI</strong></p>
</div>