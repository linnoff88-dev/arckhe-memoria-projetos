# memoria-projetos â€” ache qualquer projeto e pergunte o que jÃ¡ foi decidido

<div align="center">
  <img src="capa/logo.svg" alt="Arckhé AI" height="48" />
  <p><strong>Arckhé AI</strong></p>
</div>


[![MemÃ³ria de Projetos](guia/assets/banner.jpg)](https://inematds.github.io/memoria-projetos/guia/)

**ðŸ‡§ðŸ‡· [PortuguÃªs](README.md) Â· ðŸ‡ºðŸ‡¸ [English](README.en.md) Â· ðŸ‡ªðŸ‡¸ [EspaÃ±ol](README.es.md)**

## O que Ã©

O memoria-projetos Ã© um kit para quem tem muitos projetos e trabalha com agentes de IA, como o Claude Code e o Codex. Ele resolve dois problemas: o agente nÃ£o sabe onde estÃ¡ cada projeto e esquece o que jÃ¡ foi decidido. O comando `projeto` procura pelo que o projeto faz, nas suas pastas e no GitHub. O comando `memoria` responde perguntas como "pode fazer X?" lendo suas liÃ§Ãµes e anotaÃ§Ãµes, com data e fonte. Para usar, vocÃª precisa de um computador com Linux, Python, o Ollama e, para a memÃ³ria, a assinatura do Claude (sem chave de API). O Ã­ndice fica todo na sua mÃ¡quina; a memÃ³ria manda o texto das anotaÃ§Ãµes ao Claude para extrair os fatos. As duas metades sÃ£o independentes: dÃ¡ para instalar sÃ³ uma.

## ðŸ“– Guia de uso

Guia completo (landing + passo a passo): **https://inematds.github.io/memoria-projetos/guia/**

## As duas metades

| Metade | Pasta | Comando | Precisa de |
|---|---|---|---|
| Ãndice de projetos | `indice/` | `projeto` | Python 3, Ollama com `bge-m3`, `gh` (opcional) |
| MemÃ³ria que responde | `memoria/` | `memoria` | Python 3, servidor Hindsight, Claude Code CLI logado, Ollama com `bge-m3` |

## PrÃ©-requisitos

Instale e confira antes (o `instalar.sh` sÃ³ confere, nÃ£o instala nada do sistema):

1. **Python 3.11 ou mais novo** (`python3 --version`). Os comandos do kit usam sÃ³ a biblioteca padrÃ£o.
2. **Ollama** (https://ollama.com) rodando em `127.0.0.1:11434`, com o modelo de vetores multilÃ­ngue:
   ```bash
   ollama pull bge-m3
   ```
3. **SÃ³ para o Ã­ndice â€” GitHub CLI** (`gh`), logado em cada conta que vocÃª quer indexar:
   ```bash
   gh auth login            # repita para cada conta; o kit usa "gh auth token -u <conta>"
   gh auth status
   ```
   Sem `gh` (ou com `"contas": []`), o Ã­ndice fica sÃ³ com as pastas locais.
4. **SÃ³ para a memÃ³ria â€” Claude Code CLI** instalado e logado na sua assinatura (`claude` no PATH; o kit pÃµe `~/.npm-global/bin` e `~/.local/bin` na frente). O servidor usa o provedor `claude-code` do Hindsight, que a documentaÃ§Ã£o dele indica para **uso pessoal/local**.
5. **SÃ³ para a memÃ³ria â€” internet uma vez** na primeira subida do servidor: o PostgreSQL embutido (pg0) e o modelo de reordenaÃ§Ã£o sÃ£o baixados nessa hora. Depois, tudo roda local.
6. **systemd de usuÃ¡rio** (Linux) para os horÃ¡rios automÃ¡ticos. Para os timers rodarem mesmo sem vocÃª logado:
   ```bash
   loginctl enable-linger "$USER"
   ```
   Sem systemd, use `--sem-systemd` e rode os comandos Ã  mÃ£o (ou pelo cron).

## InstalaÃ§Ã£o

```bash
git clone https://github.com/inematds/memoria-projetos.git
cd memoria-projetos
./instalar.sh --dry-run                 # mostra tudo o que faria, sem gravar nada
./instalar.sh --instalar-hindsight      # instala as duas metades + o servidor Hindsight
```

OpÃ§Ãµes do `instalar.sh` (pode rodar quantas vezes quiser: sÃ³ grava o que mudou):

| OpÃ§Ã£o | O que faz |
|---|---|
| `--so-indice` / `--so-memoria` | instala sÃ³ uma metade |
| `--sem-systemd` | nÃ£o cria serviÃ§os nem timers |
| `--dry-run` | sÃ³ mostra o que faria |
| `--instalar-hindsight` | cria `memoria/.venv` e instala o Hindsight (â‰ˆ2 GB) |
| `--forcar` | substitui atalho ou unidade que jÃ¡ existe e nÃ£o Ã© deste kit |

O que ele faz:

- copia `config.exemplo.json` â†’ `config.json`, `sinonimos.exemplo.json` â†’ `sinonimos.json`, `.env.exemplo` â†’ `.env` e `fontes.exemplo.json` â†’ `fontes.json` **sÃ³ se ainda nÃ£o existirem** (nunca apaga config sua);
- cria os atalhos `~/.local/bin/projeto` e `~/.local/bin/memoria` apontando para esta pasta;
- escreve as unidades do systemd de usuÃ¡rio com o caminho real desta pasta, com teto de memÃ³ria (`MemoryMax`) e de tempo (`TimeoutStartSec`), `PATH` com `~/.npm-global/bin` e `~/.local/bin` na frente e timers `Persistent=true` (se a mÃ¡quina estava desligada no horÃ¡rio, roda quando ligar):

| Unidade | Quando | O que roda | Teto |
|---|---|---|---|
| `memoria-indice.timer` | todo dia 05:40 | `projeto --atualizar` | 1 GB Â· 20 min |
| `memoria-hindsight.service` | sempre ligado | servidor Hindsight em `127.0.0.1:8891` | 8 GB |
| `memoria-carga.timer` | todo dia 06:10 | `carga.py --max 25` | 512 MB Â· 60 min |
| `memoria-semana.timer` | segunda 08:00 | `semana.py --dias 7` | 512 MB Â· 30 min |

### O Hindsight, se preferir instalar Ã  mÃ£o

O `--instalar-hindsight` faz exatamente isto (versÃ£o 1.0.0, a que foi testada; a 1.2.0 jÃ¡ existe no PyPI):

```bash
python3 -m venv memoria/.venv
memoria/.venv/bin/pip install torch --index-url https://download.pytorch.org/whl/cpu   # torch sÃ³ CPU (bem menor)
memoria/.venv/bin/pip install hindsight-api==1.0.0
```

Se o `hindsight-api` jÃ¡ estiver instalado em outro lugar: `HINDSIGHT_BIN=/caminho/hindsight-api ./instalar.sh`.
TambÃ©m funciona a partir do cÃ³digo-fonte: `pip install -e "<pasta-do-clone>/hindsight-api-slim[all]"` (clone de https://github.com/vectorize-io/hindsight).
Os dados do banco ficam em `~/.pg0` (PostgreSQL embutido).

## ConfiguraÃ§Ã£o

### Ãndice â€” `indice/config.json`

| Campo | Para que serve |
|---|---|
| `raiz` | pasta onde ficam seus projetos (ex.: `~/projetos`); cada subpasta com git ou README vira um projeto |
| `contas` | contas do GitHub a indexar (`[]` = sÃ³ local). A ordem conta: em pasta com vÃ¡rios remotos, a 1Âª conta ganha |
| `subpastas_com_readme` | pastas (relativas Ã  raiz) cujas subpastas com README tambÃ©m contam como projeto (ex.: uma pasta de ferramentas) |
| `pular` / `pular_subpastas` | nomes de pasta que nunca entram |
| `saida_md` / `saida_jsonl` | onde gravar o Ã­ndice (padrÃ£o: dentro de `indice/`) |
| `ollama_url` / `ollama_modelo` | onde estÃ£o os vetores (padrÃ£o `bge-m3` local) |
| `peso_palavra` | quanto a busca por palavra soma Ã  busca por sentido (padrÃ£o 0,35) |

`sinonimos.json`: abreviaÃ§Ãµes que valem como a palavra inteira (`yt` â†’ `youtube`, `ops` â†’ `operacao`â€¦). Acrescente as suas.

### MemÃ³ria â€” `memoria/.env` e `memoria/fontes.json`

No `.env`: `HINDSIGHT_URL`, `HINDSIGHT_BANCO` (um banco por projeto ou pessoa) e a configuraÃ§Ã£o do servidor (LLM `claude-code` com `claude-haiku-5-5`, vetores `bge-m3` no Ollama, reordenaÃ§Ã£o local). A linha da chave do Ollama Ã© um texto qualquer: o Ollama nÃ£o usa chave.

No `fontes.json`, a lista do que vai para a memÃ³ria. TrÃªs tipos:

| Tipo | LÃª | Vira |
|---|---|---|
| `linhas` | itens `- ` de uma lista markdown (ex.: `LESSONS.md`) | as linhas novas do dia, em blocos de atÃ© `bloco` linhas (padrÃ£o 15) |
| `tabela` | linhas de dados de tabelas markdown (ex.: `FALHAS.md`), sem cabeÃ§alho | idem |
| `pasta-md` | cada arquivo `.md` de uma pasta (ex.: a memÃ³ria do agente) | um documento por arquivo; se o arquivo mudar, substitui o anterior |

Cada fonte tem `nome` (nÃ£o troque depois da 1Âª carga), `caminho`, `contexto` (uma frase dizendo o que Ã© aquilo) e `tag`. Em `pasta-md`, `ignorar` lista arquivos que ficam de fora (ex.: o Ã­ndice `MEMORY.md`).

**Custo da carga.** A carga sÃ³ envia o que Ã© novo ou mudou; conteÃºdo idÃªntico nem sai da mÃ¡quina. Mas cada documento enviado Ã© uma chamada ao LLM pela sua assinatura. Medido no piloto: **~5,6 mil tokens de entrada do Haiku por documento; 100 documentos â‰ˆ 8 min â‰ˆ 566 mil tokens**. Por isso o teto `--max` (padrÃ£o 25 por execuÃ§Ã£o; o que sobrar vai no dia seguinte). A 1Âª execuÃ§Ã£o Ã© a carga inicial: veja antes com `--simular` e, se quiser fazer tudo de uma vez, use `--max 200`.
AtenÃ§Ã£o: editar uma linha antiga (em `linhas` ou `tabela`) custa duas chamadas: o bloco antigo Ã© reenviado sem ela e a versÃ£o nova entra no bloco do dia. Acrescentar linhas novas nÃ£o mexe nos blocos antigos.

## Uso

```bash
# Ã­ndice
projeto --atualizar                      # refaz o Ã­ndice e os vetores (â‰ˆ1â€“2 min para mil projetos)
projeto "separa operaÃ§Ã£o do desenvolvimento"
projeto "publica vÃ­deo no youtube" -n 5  # * na nota = todas as palavras casaram

# memÃ³ria
python3 memoria/carga.py --simular       # o que seria gravado (nÃ£o envia nada)
python3 memoria/carga.py --max 200       # 1Âª carga
memoria "posso rodar processo longo sem teto de memÃ³ria?"   # resposta pronta (30â€“75 s)
memoria -r "systemd PATH"                # sÃ³ os trechos (7â€“17 s, sem LLM na resposta)
memoria --nota util "achou a regra certa"                   # avalia a Ãºltima resposta
python3 memoria/semana.py                # relatÃ³rio da semana â†’ memoria/semana-AAAA-MM-DD.md
python3 memoria/avalia.py                # grep Ã— recall Ã— reflect nas suas perguntas (gasta cota)
```

Se o Ollama estiver fora, o `projeto` cai sÃ³ na busca por palavra e avisa. Se o servidor da memÃ³ria estiver fora, o `memoria` avisa e sai com cÃ³digo 2: use o grep de sempre.

`avalia.py` usa `perguntas.jsonl` (copie de `perguntas.exemplo.jsonl` e escreva perguntas reais suas), uma por linha: `{"pergunta": "...", "palavra_grep": "palavra que vocÃª usaria no grep", "gabarito": ["regex que a resposta certa precisa ter"]}`.

## O processo, do inÃ­cio ao fim

ðŸ§  sessÃ£o (vocÃª, com ou sem o agente) Â· âš™ï¸ script/timer Â· ðŸ¤– agente

1. ðŸ§  Instalar os prÃ©-requisitos (Ollama + `ollama pull bge-m3`, `gh auth login`, Claude Code logado, `loginctl enable-linger`).
2. ðŸ§  `./instalar.sh --dry-run` e depois `./instalar.sh --instalar-hindsight`.
3. ðŸ§  Ajustar `indice/config.json` (raiz e contas), `memoria/fontes.json` (suas fontes) e `memoria/.env` (nome do banco).
4. ðŸ§  Primeiro Ã­ndice: `projeto --atualizar`; conferir com `projeto "algo que vocÃª sabe onde estÃ¡"`.
5. ðŸ§  Primeira carga da memÃ³ria: `python3 memoria/carga.py --simular`, depois `python3 memoria/carga.py --max 200`.
6. ðŸ§  Colar a regra do agente (abaixo) no `CLAUDE.md` / `AGENTS.md` dos seus projetos.
7. âš™ï¸ Todo dia 05:40: `memoria-indice.timer` refaz o Ã­ndice e calcula vetores sÃ³ do que mudou.
8. âš™ï¸ Todo dia 06:10: `memoria-carga.timer` grava na memÃ³ria sÃ³ o que Ã© novo (no mÃ¡ximo 25 documentos).
9. ðŸ¤– Na sessÃ£o, pergunta "qual projeto faz X / onde estÃ¡ Y" â†’ `projeto "X"` antes de varrer pastas.
10. ðŸ¤– Na sessÃ£o, pergunta de regra ou decisÃ£o ("pode X?", "o que decidimos sobre Y?") â†’ `memoria "â€¦"` antes do grep; depois `memoria --nota util|inutil "motivo"`.
11. âš™ï¸ Toda segunda 08:00: `memoria-semana.timer` gera `memoria/semana-AAAA-MM-DD.md` (uso, adoÃ§Ã£o pelo agente, custo, RAM).
12. ðŸ§  Ler o relatÃ³rio: ficou Ãºtil, muda a regra do agente ou desliga. De vez em quando, `python3 memoria/avalia.py` para medir a qualidade.

Desligar tudo: `systemctl --user disable --now memoria-hindsight.service memoria-carga.timer memoria-semana.timer memoria-indice.timer`.

## Fazendo o agente usar

O agente sÃ³ usa o que estÃ¡ na regra dele. Cole isto no `CLAUDE.md` (Claude Code) ou no `AGENTS.md` (Codex) dos seus projetos, ou no arquivo global de instruÃ§Ãµes:

```markdown
## Achar projeto e consultar a memÃ³ria

- Pergunta do tipo "qual projeto faz X" ou "onde estÃ¡ Y": rode `projeto "X"` ANTES de varrer pastas
  ou o histÃ³rico. NÃ£o leia o projetos.md inteiro; use o comando ou `grep -i`.
- Pergunta de regra, decisÃ£o ou erro jÃ¡ visto ("pode X?", "o que decidimos sobre Y?",
  "que erro jÃ¡ deu com Z?"): rode `memoria "pergunta"` (resposta pronta) ou `memoria -r "palavras"`
  (sÃ³ trechos, mais rÃ¡pido) ANTES do grep nas liÃ§Ãµes e falhas.
- Depois de usar a resposta, avalie: `memoria --nota util "motivo"` ou `memoria --nota inutil "motivo"`.
- Se `memoria` sair com cÃ³digo 2 (servidor fora), siga com o grep de sempre e avise.
```

NÃ£o coloque a resposta da memÃ³ria no inÃ­cio de toda sessÃ£o: o agente pergunta quando precisa. Assim o comeÃ§o da sessÃ£o fica sempre igual (o cache funciona) e a cota sÃ³ Ã© gasta quando hÃ¡ pergunta.

## CrÃ©ditos

- A memÃ³ria usa o **Hindsight**, da vectorize-io, licenÃ§a MIT: https://github.com/vectorize-io/hindsight
- Espelho e guia em portuguÃªs: https://github.com/inematds/hindsight Â· https://inematds.github.io/hindsight/guia/
- Vetores: modelo `bge-m3` (BAAI) servido pelo Ollama.

## LicenÃ§a

MIT â€” veja [LICENSE](LICENSE). Â© 2026 INEMA.

---

<div align="center">
  <img src="capa/logo.svg" alt="Arckhé AI" height="32" />
  <p><strong>Arckhé AI</strong></p>
</div>