name: initial
layout: true
class: center, middle
---
#Ansible
[no dia a dia]
.footnote[github: [ansible no dia a dia](https://github.com/tatsuryu/ansible)]
---
layout: false
.left-column[
  ## Quem sou eu?
]
.right-column[
    Ícaro
  
- Mais de 15 anos de experiência em administração de ambientes Unix, com foco em infraestrutura. 
  
- Instrutor de cursos preparatóros para ICND1 e ICND2 (Academia CISCO). 

- Atuei em provedores, e prestando consultoria para provedores. 

- Formado em Processamento de dados pela ETE Fernando Prestes Sorocaba/SP. 

- Autodidata, e usuário de Gentoo/Funtoo.

- Trabalho atualmente na [ISPTI](https://www.ispti.com.br)

]
---
.left-column[
  ## Quem sou eu?
  ## Contexto
]
.right-column[
  Problemas ao gerir muitos servidores:

- sem padronização nas instalações

- sem documentação

- alterações em massa trabalhosas e propensas a erro

- atualizações beirando o impossível

]
---
.left-column[
  ## Quem sou eu ?
  ## Contexto
  ## O que é ?
]
.right-column[
  Ansible é:

- Gerenciador de configuração

- Ferramenta de automação

- Ferramenta de deploy

É escrito em python, é _client-less_, e roda em muitos sistemas _Unix-like_ e em Windows.
Utiliza o formato declarativo: [YAML](https://pt.wikipedia.org/wiki/YAML) no seu sistema de configuração.
]
---
.left-column[
  ## Quem sou eu ?
  ## Contexto
  ## O que é ?
  ## Instalação
]
.right-column[
  Forma de instalação recomendada é utilizando o pip, e de preferência com um virtualenv ou somente para seu usuário:

- Instalação utilizando o virtualenv
```
python3 -mvenv ansible
source ansible/bin/activate
pip install ansible
```

- Instalação limitada ao usuário:
```
pip install --user ansible
```
]
---
.left-column[
  ## Quem sou eu ?
  ## Contexto
  ## O que é ?
  ## Instalação
  ## Primeiros passos
]
.right-column[
  A configuração dos hosts é feito no arquivo de [_inventory_](https://docs.ansible.com/ansible/2.4/intro_inventory.html) que permite vários formatos como: json(dinâmico), ini e yaml.

- Exemplo:

```
[vagrant]
stretch_vagrant ansible_host=127.0.0.1 ansible_port=2222 ansible_user=vagrant ansible_ssh_private_key_file=.vagrant/machines/default/virtualbox/private_key


[docker]
docker_host ansible_connection=docker ansible_host=testing ansible_user=root
```

]