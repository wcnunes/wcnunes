<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Minhas Tecnologias GitHub</title>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/devicon@2.15.1/devicon.min.css">
  <style>
    body { background: #151515; color: #fff; font-family: sans-serif; }
    .icon-grid { display: flex; flex-wrap: wrap; gap: 24px; justify-content: center; }
    .icon-card {
      background: #222;
      border-radius: 12px;
      box-shadow: 0 2px 16px #0006;
      padding: 18px;
      display: flex;
      flex-direction: column;
      align-items: center;
      transition: transform 0.3s, box-shadow 0.3s;
      cursor: pointer;
    }
    .icon-card:hover {
      transform: scale(1.13) rotate(-6deg);
      box-shadow: 0 8px 32px #0ff7;
      background: linear-gradient(135deg, #222, #0ff2);
    }
    .icon-card i { font-size: 3rem; margin-bottom: 10px; }
    .icon-label { font-size: 1.2rem; margin-top: 4px; text-shadow: 0 2px 8px #000a;}
  </style>
</head>
<body>
  <h1>Minhas Tecnologias GitHub</h1>
  <div id="iconGrid" class="icon-grid"></div>
  <script>
    // Seu username do GitHub
    const username = "w1llsystems";

    // Mapeamento de linguagens/ferramentas para classes do DEVICON
    const deviconMap = {
      "JavaScript": "devicon-javascript-plain colored",
      "TypeScript": "devicon-typescript-plain colored",
      "Python": "devicon-python-plain colored",
      "HTML": "devicon-html5-plain colored",
      "CSS": "devicon-css3-plain colored",
      "Java": "devicon-java-plain colored",
      "Ruby": "devicon-ruby-plain colored",
      "PHP": "devicon-php-plain colored",
      "C++": "devicon-cplusplus-plain colored",
      "C": "devicon-c-plain colored",
      "Go": "devicon-go-plain colored",
      "Rust": "devicon-rust-plain colored",
      "Shell": "devicon-bash-plain colored",
      "Docker": "devicon-docker-plain colored",
      "Kubernetes": "devicon-kubernetes-plain colored",
      "Node.js": "devicon-nodejs-plain colored",
      "React": "devicon-react-original colored",
      "Vue.js": "devicon-vuejs-plain colored",
      "Angular": "devicon-angularjs-plain colored",
      "Next.js": "devicon-nextjs-original colored",
      "MongoDB": "devicon-mongodb-plain colored",
      "MySQL": "devicon-mysql-plain colored",
      "PostgreSQL": "devicon-postgresql-plain colored",
      "Git": "devicon-git-plain colored",
      "GitHub": "devicon-github-original colored"
      // Adicione mais mapeamentos conforme necessário!
    };

    // Busca os repositórios e suas linguagens
    async function fetchTechs() {
      // 1. Pega os repositórios
      const repos = await fetch(`https://api.github.com/users/${username}/repos?per_page=100`)
        .then(res => res.json());

      // 2. Extrai as linguagens principais
      const langsSet = new Set();
      for (const repo of repos) {
        if (repo.language && deviconMap[repo.language]) {
          langsSet.add(repo.language);
        }
        // Para mais profundidade, poderia buscar /languages de cada repo
      }

      // 3. Exemplo: busca as linguagens de cada repo (mais detalhado)
      // const langDetails = await Promise.all(repos.map(r => fetch(r.languages_url).then(r=>r.json())));
      // langDetails.forEach(obj => Object.keys(obj).forEach(lang => langsSet.add(lang)));

      // 4. Adicione manualmente ferramentas se quiser
      ["GitHub", "Git", "Node.js", "Docker"].forEach(tool=>langsSet.add(tool));

      return Array.from(langsSet);
    }

    // Renderiza os ícones
    async function renderIcons() {
      const techs = await fetchTechs();
      const grid = document.getElementById('iconGrid');
      for (const tech of techs) {
        const iconClass = deviconMap[tech];
        if (iconClass) {
          grid.innerHTML += `
            <div class="icon-card" title="${tech}">
              <i class="${iconClass}"></i>
              <span class="icon-label">${tech}</span>
            </div>
          `;
        }
      }
    }

    renderIcons();
  </script>
</body>
</html>
