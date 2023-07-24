# Replataform Wordpress - AWS

> Status da última build <br>
> ![](https://codebuild.us-east-1.amazonaws.com/badges?uuid=eyJlbmNyeXB0ZWREYXRhIjoiYXBJWXpBTzF6cTZnNWt1T2oxcVl0TGQraVFxdk9kRVFHUHlJYmhGeWxKcDVqQVB6a2FaWXJOaGk1YmM4TXFuOEpqcitwalU3cDRUQ1gyQldtVDVhTXY0PSIsIml2UGFyYW1ldGVyU3BlYyI6ImZWUU5qbG5FSHJQN200UzkiLCJtYXRlcmlhbFNldFNlcmlhbCI6MX0%3D&branch=main)

Bem vindo(a) a documentação do desafio final do Ignite Brasil.

Neste documento, apresento a documentação da replataforma de um site WordPress utilizando infraestrutura na <a href="https://aws.amazon.com/" target="_blank"> AWS </a>, com o uso de ferramentas como o <a href="https://www.terraform.io/" target="_blank">Terraform</a>, <a href="https://helm.sh/" target="_blank">Helm</a> e <a href="https://www.ansible.com/" target="_blank"> Ansible</a>.

## TLDR

Este guia aborda a replataforma de um site WordPress, com a utilização de infraestrutura na AWS através do Terraform, instalação de charts com Helm e configuração com Ansible. Também destaca os pontos negativos associados ao uso do WordPress, como a arquitetura monolítica e desafios de segurança e desempenho.

Ao adotar boas práticas de desenvolvimento, otimização e segurança, podemos superar esses desafios e criar um ambiente WordPress moderno e eficiente.

## Ferramentas Adcionais

- <a href="https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html" target="_blank">AWS CLI</a>
- <a href="https://kubernetes.io/docs/tasks/tools/" target="_blank">Kubectl</a>

### Pré requisitos

- Chave de acesso programático para AWS CLI
- 

## O que é IaC?

IaC (Infrastructure as Code), ou Infraestrutura como Código, é uma abordagem que permite gerenciar e provisionar a infraestrutura do projeto por meio de código, em vez de configurações manuais. Com IaC, é possível descrever a infraestrutura desejada em arquivos de código, como o Terraform, e então implantar e gerenciar essa infraestrutura de forma automatizada e escalável.

## Etapas

- **Infraestrutura na AWS com Terraform**: Utilizando a ferramenta Terraform, posso definir e provisionar a infraestrutura necessária na AWS de forma automatizada. Isso inclui a criação de instâncias EC2, configuração de redes, balanceadores de carga, entre outros recursos.
- **Instalação de Charts com Helm**: O Helm é um gerenciador de pacotes para Kubernetes que permite instalar, atualizar e gerenciar aplicações de forma padronizada. No contexto da minha replataforma do WordPress, podemos utilizar o Helm para implementar os componentes necessários, e um *bootstrap* do site já com banco de dados.

- **Configuração com Ansible**: O Ansible é uma ferramenta de automação que me permite configurar e gerenciar a infraestrutura de forma declarativa. Com o Ansible, é possível definir as configurações desejadas para o ambiente do WordPress, como ajustes de segurança, configuração do servidor web, instalação de plugins e temas, entre outros.

## Layout do projeto

Abaixo está a arquiterura geral:

<div style="width: 640px; height: 480px; margin: 10px; position: relative;"><iframe allowfullscreen frameborder="0" style="width:640px; height:480px" src="https://lucid.app/documents/embedded/53448e69-6a1f-43d0-a8a8-d0cbe9418881" id="LZ8_Yo9J0Fpn"></iframe></div>

## Pontos Positivos
**Fácil de usar e personalizar**: O WordPress é conhecido por sua interface intuitiva e amigável, tornando-o acessível para usuários iniciantes e experientes. Com um editor de arrastar e soltar (Gutenberg) e uma vasta biblioteca de temas e plugins, você pode personalizar facilmente a aparência e as funcionalidades do seu site sem precisar de habilidades avançadas de desenvolvimento.

**Comunidade e suporte**: O WordPress possui uma comunidade enorme e ativa de desenvolvedores, designers e usuários. Isso significa que você pode encontrar facilmente soluções para problemas, obter dicas úteis e até mesmo ter acesso a um vasto número de recursos gratuitos, como temas, plugins e tutoriais.

**SEO-friendly**: O WordPress foi criado com o foco em otimização para mecanismos de busca (SEO). Ele oferece várias opções e plugins para melhorar o desempenho do seu site nos resultados de pesquisa, como a criação de URLs amigáveis, otimização de títulos e meta descrições, integração com ferramentas de análise e muito mais.

**Flexibilidade e escalabilidade**: O WordPress é altamente flexível e pode ser usado para criar diversos tipos de sites, desde blogs pessoais e sites de pequenas empresas até grandes portais de notícias e lojas online. Sua estrutura modular permite que você adicione novas funcionalidades à medida que seu site cresce, tornando-o escalável para acompanhar suas necessidades em expansão.

## Pontos Negativos

Embora o WordPress seja uma plataforma popular e amplamente utilizada para a criação de sites, é importante estar ciente de alguns pontos negativos associados ao seu uso, especialmente ao "replataformá-lo" em um ambiente mais moderno:

- **Arquitetura Monolítica**: O WordPress foi projetado como uma aplicação monolítica, o que significa que todos os componentes, como o servidor web, banco de dados e plugins, estão fortemente acoplados. Isso pode dificultar a escalabilidade e flexibilidade da aplicação em ambientes mais complexos.

- **Segurança**: Devido à sua popularidade, o WordPress é frequentemente alvo de ataques cibernéticos. É essencial tomar medidas adicionais para fortalecer a segurança do ambiente, como a aplicação de atualizações de segurança, configurações adequadas e uso de plugins confiáveis.

- **Desempenho**: Sites WordPress mal otimizados podem sofrer com problemas de desempenho, como tempos de carregamento lentos. É fundamental adotar boas práticas de otimização, como o uso de cache, compactação de recursos e otimização de consultas ao banco de dados, para garantir um desempenho adequado.

- **Dependência de Plugins**: O WordPress é conhecido por sua vasta biblioteca de plugins, que adicionam funcionalidades extras ao site. No entanto, o uso excessivo de plugins pode tornar o ambiente mais complexo e aumentar o risco de incompatibilidades, vulnerabilidades de segurança e problemas de desempenho.

## Commands

- `mkdocs new [dir-name]` - Create a new project.
- `mkdocs serve` - Start the live-reloading docs server.
- `mkdocs build` - Build the documentation site.
- `mkdocs -h` - Print help message and exit.

## Commands

    mkdocs.yml    # The configuration file.
    docs/
        index.md  # The documentation homepage.
        ...       # Olá Laix.
