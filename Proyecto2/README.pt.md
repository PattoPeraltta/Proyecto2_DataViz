# 🎵 SvelteKit Vinyl Music Player

Um player de música interativo e visualmente rico construído com SvelteKit. Navegue por uma coleção de álbuns icônicos, veja seus detalhes e ouça faixas de amostra — tudo apresentado como discos de vinil animados.

## 🚀 Funcionalidades

- **Tela de Boas-Vindas Animada**: Recebe os usuários com um efeito de máquina de escrever.
- **Tocador de Disco de Vinil**: Álbuns visualizados como vinis girando, com sobreposições de gênero, época e popularidade.
- **Carrossel de Álbuns**: Navegue e selecione entre uma coleção de álbuns.
- **Informações Detalhadas do Álbum**: Veja artista, ano, gênero, ranking, descrição e lista de faixas.
- **Reprodução de Áudio**: Controles de play, pause e stop para cada álbum.
- **Design Responsivo**: Funciona perfeitamente em desktop e dispositivos móveis.
- **Baseado em Dados**: Dados dos álbuns carregados de um arquivo CSV para fácil atualização.

## 🗂️ Estrutura do Projeto

```
Proyecto2/
├── public/              # Arquivos estáticos (imagens, ícones)
├── src/
│   ├── routes/
│   │   ├── +layout.svelte   # Layout do aplicativo
│   │   └── +page.svelte     # UI principal do player & lógica
│   └── lib/                 # (Opcional) Componentes/utilitários compartilhados
├── static/
│   ├── covers/          # Imagens das capas dos álbuns
│   ├── data/albums.csv  # Metadados dos álbuns (id, título, artista, ...)
│   ├── music/           # Arquivos de áudio de cada álbum
│   ├── overlays/        # Sobreposições de UI (época, sulcos, ranking)
│   └── vinyls/          # Imagens base de vinil por gênero
├── package.json         # Metadados & scripts do projeto
├── svelte.config.js     # Configuração do SvelteKit
└── vite.config.ts       # Configuração do Vite
```

## 📦 Instalação & Desenvolvimento

1. **Instale as dependências:**
   ```bash
   npm install
   # ou
   pnpm install
   # ou
   yarn install
   ```

2. **Execute o servidor de desenvolvimento:**
   ```bash
   npm run dev
   # ou
   npm run dev -- --open  # Abre no navegador
   ```

3. **Build para produção:**
   ```bash
   npm run build
   ```

4. **Preview do build de produção:**
   ```bash
   npm run preview
   ```

## 📝 Dados & Personalização

- **Adicionar/Atualizar Álbuns:**
  - Edite `static/data/albums.csv` para adicionar novos álbuns ou atualizar existentes.
  - Coloque as imagens das capas em `static/covers/` e os arquivos de áudio em `static/music/`.
- **Gêneros, Épocas e Sobreposições:**
  - Adicione ou modifique imagens base de vinil em `static/vinyls/`.
  - Sobreposições de época e sulcos estão em `static/overlays/`.

## 🛠️ Scripts

- `npm run dev` — Inicia o servidor de desenvolvimento
- `npm run build` — Build para produção
- `npm run preview` — Preview do build de produção
- `npm run lint` — Lint no código
- `npm run format` — Formata o código

## 📚 Tecnologias Utilizadas

- [SvelteKit](https://kit.svelte.dev/)
- [Tailwind CSS](https://tailwindcss.com/)
- [Vite](https://vitejs.dev/)

## 📄 Licença

MIT (ou especifique sua licença aqui)

---

> Inspirado pela beleza do vinil e o poder do Svelte.
