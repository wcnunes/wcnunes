import os
import requests

GITHUB_USERNAME = "wcnunes"  # Troque se quiser analisar outro perfil
GITHUB_TOKEN = os.getenv("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")  # Pegue o token da variável de ambiente

if not GITHUB_TOKEN:
    raise Exception("Varíavel de ambiente GITHUB_TOKEN não definida. Exporte o token antes de rodar o script.")

def get_repos(user):
    url = f"https://api.github.com/users/{user}/repos?per_page=100"
    headers = {"Authorization": f"token {GITHUB_TOKEN}"}
    response = requests.get(url, headers=headers)
    response.raise_for_status()
    return response.json()

def get_languages(repo):
    url = repo["languages_url"]
    headers = {"Authorization": f"token {GITHUB_TOKEN}"}
    response = requests.get(url, headers=headers)
    response.raise_for_status()
    return response.json()

def main():
    repos = get_repos(GITHUB_USERNAME)
    language_count = {}

    for repo in repos:
        languages = get_languages(repo)
        for lang in languages:
            language_count[lang] = language_count.get(lang, 0) + languages[lang]

    # Pega as 8 tecnologias mais usadas
    top_languages = sorted(language_count, key=language_count.get, reverse=True)[:8]

    # Gera markdown com ícones
    print("Cole este bloco no seu README.md:\n")
    print("## Tecnologias e Ferramentas Principais\n")
    icons = []
    shields_lang = {
        "Python": "python",
        "JavaScript": "javascript",
        "TypeScript": "typescript",
        "HTML": "html5",
        "CSS": "css3",
        "Shell": "gnubash",
        "C++": "cplusplus",
        "C": "c",
        "Java": "java",
        "Go": "go",
        "Ruby": "ruby"
        # Adicione mais mapeamentos se precisar
    }
    for lang in top_languages:
        icon_name = shields_lang.get(lang, lang.lower())
        icon_md = f"![{lang}](https://img.shields.io/badge/-{lang}-black?style=flat-square&logo={icon_name})"
        icons.append(icon_md)
    print(" ".join(icons))

if __name__ == "__main__":
    main()
