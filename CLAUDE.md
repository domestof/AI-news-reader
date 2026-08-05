# AI-news-reader

Aquest repositori **no conté codi d'aplicació**. El projecte és una rutina
programada al núvol (Claude Code Routine) que genera cada dia un resum de
notícies d'IA/tecnologia a partir dels correus de Gmail de l'usuari (LinkedIn,
TLDR, Substack), i el deixa com a esborrany a Gmail.

- **Arquitectura completa, pipeline i configuració necessària**:
  [`docs/ARQUITECTURA.md`](docs/ARQUITECTURA.md)
- **Historial de decisions i incidències resoltes** (permisos MCP, token OAuth
  caducat, bloqueig de xarxa de Telegram, etc.): [`docs/ESTAT.md`](docs/ESTAT.md)

## Referència ràpida

- Rutina: "Resum diari IA/Tech (LinkedIn + newsletters)" —
  veure-la a https://claude.ai/code/routines
- Horari: `0 5 * * *` UTC (7:00 Madrid en horari d'estiu)
- Gestió: eina `RemoteTrigger` (`action: get|update|run`) o skill `schedule`
- Estat: **funcionant** (confirmat 2026-08-05) — l'usuari ha de prémer "Enviar"
  manualment cada dia sobre l'esborrany que genera la rutina

Abans de tocar la rutina (afegir fonts, canviar horari, límits, etc.), llegeix
`docs/ARQUITECTURA.md` per entendre les limitacions dels connectors (Gmail només
crea esborranys, Drive només crea fitxers nous, Telegram bloquejat per xarxa en
aquest entorn) abans de proposar canvis que en depenguin.
