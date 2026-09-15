# jaylog-book

Documentação guiada da biblioteca [jaylog](https://github.com/Gpocas/jaylog), construída com [Zensical](https://zensical.org) e publicada no GitHub Pages.

## Desenvolvimento local

```bash
uv sync
uv run zensical serve
```

Isso abre a documentação em `http://localhost:8000` com recarregamento automático a cada alteração em `docs/`.

## Build

```bash
uv run zensical build --clean
```

O site estático é gerado em `site/`.

## Publicação

O workflow em `.github/workflows/docs.yml` publica automaticamente no GitHub Pages a cada push para `main`. Habilite o GitHub Pages do repositório com a fonte "GitHub Actions" em Settings → Pages.
