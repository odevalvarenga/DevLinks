# DevLinks

Página de links pessoais criada a partir de um layout do Figma, com apoio de um agente de programação com IA. As orientações usadas pelo agente estão em [`AGENTS.md`](./AGENTS.md).

---

## O que o projeto oferece

- Perfil com avatar emoldurado por um degradê e identificação do usuário
- Controle para alternar entre os temas claro e escuro
- Armazenamento da escolha de tema no **localStorage**, preservado entre visitas
- Uso da preferência de cores do sistema na primeira visita (`prefers-color-scheme`)
- Aplicação do tema por um script em `index.html` antes da inicialização do React, evitando a exibição rápida do tema incorreto
- Botões de links com aparência translúcida inspirada em *glassmorphism*
- Ícones para GitHub, Instagram, YouTube e LinkedIn
- Plano de fundo com degradês desfocados que mudam conforme o tema
- Apresentação adaptável a celulares e computadores
- Recursos de acessibilidade, incluindo `role="switch"`, `aria-checked`, `aria-label` e indicação de foco pelo teclado

## Tecnologias utilizadas

| Tecnologia | Finalidade |
| --- | --- |
| [React 19](https://react.dev) | Construção da interface usando componentes, Props e Hooks |
| [Vite 7](https://vite.dev) | Execução local e geração da versão de produção |
| [Tailwind CSS 4](https://tailwindcss.com) | Estilos, integrado pelo plugin `@tailwindcss/vite` |
| [Playwright](https://playwright.dev) | Testes de ponta a ponta em telas desktop e mobile |
| [Context7 MCP](https://github.com/upstash/context7) | Consulta à documentação atual das bibliotecas pelo agente de IA |
| [Playwright MCP](https://github.com/microsoft/playwright-mcp) | Abertura e verificação da página em um navegador pelo agente |

## Executar localmente

É necessário ter o **Node.js 20.19 ou superior** (ou a versão 22 ou superior).

```bash
# 1. Baixe as dependências do projeto
npm install

# 2. Inicie o servidor de desenvolvimento
npm run dev
# Abra http://localhost:5173

# 3. Crie e visualize a versão de produção
npm run build
npm run preview
```

## Testes com Playwright

```bash
# Instale o Chromium na primeira execução
npx playwright install chromium

# Execute a suíte; o servidor Vite é iniciado automaticamente
npm run test:e2e

# Abra a interface interativa dos testes
npm run test:e2e:ui

# Consulte o relatório HTML mais recente
npx playwright show-report
```

A suíte cobre os perfis **desktop-chromium** e **mobile-chromium** (Pixel 7). Ela confere:

1. Exibição do perfil, dos quatro links, das redes sociais e do rodapé
2. Abertura dos links externos em outra aba e uso de `rel="noopener noreferrer"`
3. Inicialização no tema escuro quando o sistema está configurado nesse modo
4. Alternância entre os temas claro e escuro
5. Persistência da preferência no `localStorage`, inclusive após recarregar a página
6. Desconsideração de valores inválidos armazenados para o tema
7. Ausência de rolagem horizontal em diferentes larguras de tela

## Organização dos arquivos

```
devLinks/
├── public/
│   └── avatar.svg              # Avatar; troque pelo seu
├── src/
│   ├── components/
│   │   ├── Background.jsx      # Degradês usados no fundo
│   │   ├── Footer.jsx          # Rodapé com autoria
│   │   ├── LinkButton.jsx      # Botão reutilizável para um link
│   │   ├── LinkList.jsx        # Conjunto de botões de link
│   │   ├── Profile.jsx         # Avatar e nome de usuário
│   │   ├── SocialIcon.jsx      # Ícones SVG das redes sociais
│   │   ├── SocialLinks.jsx     # Grupo de ícones sociais
│   │   └── ThemeToggle.jsx     # Controle dos temas claro e escuro
│   ├── context/
│   │   └── ThemeContext.jsx    # Contexto que disponibiliza o tema
│   ├── hooks/
│   │   ├── useLocalStorage.js  # Estado sincronizado com o armazenamento local
│   │   └── useTheme.js         # Acesso ao tema
│   ├── data/
│   │   └── profile.js          # Conteúdo editável: perfil, links e redes
│   ├── App.jsx
│   ├── index.css               # Tailwind e variante dark baseada em classe
│   └── main.jsx
├── tests/
│   └── devlinks.spec.js        # Testes de ponta a ponta com Playwright
├── mcp-config/
│   ├── mcp.json                # Configuração dos MCPs para copiar para .mcp.json
│   └── vscode-mcp.json         # Configuração dos MCPs para o Copilot
├── AGENTS.md                   # Regras e contexto para agentes de IA
├── index.html
├── package.json
├── playwright.config.js
└── vite.config.js
```

## Recursos de React no projeto

### Componentes e Props

A interface é dividida em componentes que recebem os dados por **props**, mantendo textos e endereços fora da implementação visual:

```jsx
<Profile avatar={profile.avatar} alt={profile.avatarAlt} username={profile.username} />
<LinkList links={links} />
<SocialLinks items={socials} />
<Footer author={footer.author} />
```

O mesmo `LinkButton` atende todos os itens. Para incluir um novo botão, adicione um objeto ao array `links` em `src/data/profile.js`.

### Hooks

- **`useState` e `useEffect`**: usados em `useLocalStorage` para manter um valor e atualizá-lo quando houver mudanças.
- **`useContext`**: utilizado por `useTheme` para acessar o tema sem repassar props por vários componentes.
- **`useCallback` e `useMemo`**: ajudam a manter a função `toggleTheme` e o objeto de contexto entre renderizações.
- **Hooks próprios**: `useLocalStorage`, que pode guardar diferentes tipos de valor, e `useTheme`, dedicado ao tema.

### Persistência do tema

1. O `ThemeProvider` procura o tema salvo na chave `devlinks:theme`; sem um valor salvo, considera a preferência do sistema.
2. Ao mudar o tema, um `useEffect` adiciona ou remove a classe `dark` do elemento `<html>`.
3. No Tailwind 4, a variante `dark:` acompanha essa classe:
   ```css
   @custom-variant dark (&:where(.dark, .dark *));
   ```
4. O hook `useLocalStorage` grava a nova escolha automaticamente.

## Alterar os dados do perfil

Atualize o arquivo **`src/data/profile.js`**:

```js
export const profile = { username: '@seu.perfil', avatar: '/avatar.svg', avatarAlt: 'Foto de perfil' }
export const links = [{ id: 'ebook', label: 'Baixe meu e-book', href: 'https://...' }, /* ... */]
export const socials = [{ network: 'github', label: 'GitHub', href: 'https://github.com/...' }, /* ... */]
export const footer = { author: 'seu nome' }
```

## Inteligência artificial no desenvolvimento

O desenvolvimento contou com um agente de programação com IA, orientado pelas regras registradas em [`AGENTS.md`](./AGENTS.md). Foram utilizados dois servidores MCP:

- **Context7**: consulta referências atuais de React 19, Tailwind CSS 4, Vite 7 e Playwright, ajudando a evitar APIs antigas — como a configuração `tailwind.config.js` com `darkMode: 'class'` do Tailwind 3.
- **Playwright MCP**: permite abrir a aplicação no navegador, interagir com o controle de tema e revisar a apresentação da página.
