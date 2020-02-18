name: initial
layout: true
class: center, middle
---
#Ansible
no dia a dia
.footnote[github: [ansible no dia a dia](https://github.com/tatsuryu/ansible_dia_a_dia)]
---
layout: false
.left-column[
  ## Quem sou eu ?
]
.right-column[
    Ícaro
  
- Mais de 15 anos de experiência em administração de ambientes Unix, com foco em infraestrutura. 
  
- Instrutor de cursos preparatóros para ICND1 e ICND2 (Academia CISCO). 

- Atuei em provedores, e prestando consultoria para provedores. 

- Formado em Processamento de dados pela ETE Fernando Prestes Sorocaba/SP. 

- Autodidata, e usuário de Gentoo/Funtoo.

- Trabalho atualmente na [ISPTI](https://www.ispti.com.br)

.footnote[contatos: github[/tatsuryu](https://github.com/tatsuryu), telegram: [IcaroTatsu](https://t.me/IcaroTatsu)]
]
---
.left-column[
  ## Quem sou eu ?
  ## Contexto
]
.right-column[
### Problemas ao gerir muitos servidores:
<br><br>
- não há padronização nas instalações

- documentação defasada, incompleta ou ausente

- muitos _shellscripts_ que sempre aumentam em tamanho e complexidade pra prever todos os casos

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

<div align="center"><img width="30%" src="https://cdn.worldvectorlogo.com/logos/ansible.svg"></img></div>

<br>
É escrito em python, é _client-less_, e roda em muitos sistemas _Unix-like_ e em Windows.
Utiliza o formato declarativo: [YAML](https://pt.wikipedia.org/wiki/YAML) no seu sistema de configuração.

<br>
- Gerenciador de configuração

- Ferramenta de automação

- Ferramenta de deploy

]
---
.left-column[
  ## Quem sou eu ?
  ## Contexto
  ## O que é ?
  ## Instalação
]
.right-column[
- Instalação utilizando o virtualenv
```
python3 -mvenv ansible
source ansible/bin/activate
pip install wheel setuptools
pip install ansible
```
<div><asciinema-player src="./screencasts/install_virtual.cast"></asciinema-player></div>
]
---
.left-column[
  ## Quem sou eu ?
  ## Contexto
  ## O que é ?
  ## Instalação
]
.right-column[
  - Instalação limitada ao usuário:
```
pip install --user ansible
```
<br><br>
<div><asciinema-player src="./screencasts/install.cast"></asciinema-player></div>
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

Exemplo:
- Subindo os hosts:
```sh
~$ docker run --name=testing_stretch -d tatsuryu/debian-systemd-molecule:9
~$ vagrant init debian/stretch64
~$ vagrant up
```

- _Inventory_:

```
localhost ansible_connection=local

[vagrant]
stretch_vagrant ansible_host=127.0.0.1 ansible_port=2222 ansible_user=vagrant ansible_ssh_private_key_file=.vagrant/machines/default/virtualbox/private_key


[docker_containers]
docker_host ansible_connection=docker ansible_host=testing ansible_user=root
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
- Executando ansible: 
```
~$ ansible all -i inventory -m setup -a "filter=ansible_virtualization_type"
stretch_vagrant | SUCCESS => {
    "ansible_facts": {
        "ansible_virtualization_type": "virtualbox",
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false
}
docker_host | SUCCESS => {
    "ansible_facts": {
        "ansible_virtualization_type": "docker",
        "discovered_interpreter_python": "/usr/bin/python3.5"
    },
    "changed": false
}
localhost | SUCCESS => {
    "ansible_facts": {
        "ansible_virtualization_type": "kvm",
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false
}
```
]
---
.left-column[
  ## Quem sou eu ?
  ## Contexto
  ## O que é ?
  ## Instalação
  ## Primeiros passos
  ## Idempotência
]
.right-column[
  ### Idempotência: 
  É a propriedade de que algumas operações têm de poderem ser aplicadas várias vezes sem que o valor do resultado se altere após a aplicação inicial.

  - Primeira execução:

```sh
~$ docker run --privileged -v /sys/fs/cgroup:/sys/fs/cgroup:ro --name=testing -d tatsuryu/debian-systemd-molecule:9
~$ cat inventory 
testing ansible_connection=docker ansible_user=root
~$ ansible testing -i inventory -m apt -a "update_cache=yes name=apache2"
testing | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.5"
    },
    "cache_update_time": 1582028646,
    "cache_updated": false,
    "changed": true,
    "stderr": "debconf: delaying package configuration, since apt-utils is not installed\n",
    "stderr_lines": [
        "debconf: delaying package configuration, since apt-utils is not installed"
    ],
    "stdout": ...,
    "stdout_lines": [ ... ],
}
```
]
---
.left-column[
  ## Quem sou eu ?
  ## Contexto
  ## O que é ?
  ## Instalação
  ## Primeiros passos
  ## Idempotência
]
.right-column[

  - Segunda execução:

```
~$ ansible testing -i inventory -m apt -a "update_cache=yes name=apache2"
testing | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.5"
    },
    "cache_update_time": 1582028646,
    "cache_updated": false,
    "changed": false
}
```
]
---
.left-column[
  ## Playbooks
]
.right-column[

  Escritos em **YAML**, descrevem o _estado_ que você deseja.

  Exemplo:

  ```
  ~$ cat apache.yaml
  ---
  - hosts: all
    tasks:
      - name: Instala apache
        apt:
          update_cache: yes
          name: apache2
          state: present
  ...
  ```

  Executando:

  ```sh
~$ ansible-playbook -i inventory apache.yaml --limit testing

PLAY [all] **********************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************
ok: [testing]

TASK [Instala apache] ***********************************************************************************************
ok: [testing]

PLAY RECAP **********************************************************************************************************
testing                    : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
  ```
]
---
.left-column[
  ## Playbooks
  ## YAML
]
.right-column[
## YAML Ain't Markup Language

- Inicialmente: _Yet Another Markup Language_
- Foi projetada para representar de forma visualmente simples
- Representa dados como uma combinação: listas, hashes e dados escalares

Exemplo:
```
---
evento:
  nome: "Meetup DevCia"
  topicos:
    - ansible
    - "JAMStack com GatsbyJS"
...
```
]
---
.left-column[
  ## Playbooks
  ## YAML
]
.right-column[
```
~$ cat nada.yaml
---
nome: Teste
tipo: exemplo
dados: |
  alguns valores em
  multiplas linhas.
lista:
  - item 1
  - item 2
  - item 3
conteudo: >
  isto é um multi-line
  formatting
...
```

```
~$ python -c 'import json; import yaml; x=yaml.safe_load(open("nada.yaml","r").read()); print(json.dumps(x, indent=2))'
{
  "nome": "Teste",
  "tipo": "exemplo",
  "dados": "alguns valores em\nmultiplas linhas.\n",
  "lista": [
    "item 1",
    "item 2",
    "item 3"
  ],
  "conteudo": "isto \u00e9 um multi-line formatting\n"
}
```
]
---
.left-column[
  ## Playbooks
  ## YAML
  ## Obtendo ajuda sobre os módulos
]
.right-column[
Para obter uma lista dos módulos do ansible, você pode acessar a [página de módulos](https://docs.ansible.com/ansible/latest/modules/modules_by_category.html), ou localmente via shell:
```
~$ ansible-doc -l
```
Para obter ajuda sobre um módulo específico é só informar o nome:
```
~$ ansible-doc apt
```
]
---
.left-column[
  ## Playbooks
  ## YAML
  ## Obtendo ajuda sobre os módulos
  ## Handlers
]
.right-column[
## Handlers 
São uma resposta a uma alteração feita por um evento. Pode ser utilizado por exemplo quando é feita alguma alteração em uma configuração e você precisa que o serviço seja reiniciado.

```
  ---
  - hosts: all
    handlers:
      - name: Reinicia apache
        service:
          name: apache2
          state: restarted
    tasks:
      - name: Instala apache
        apt:
          update_cache: yes
          name: apache2
          state: present
      
      - name: Copia configuração do site
        copy:
          src: site.conf
          dest: /etc/apache2/sites-enabled/002-site.conf
        notify: Reinicia apache
  ...
```
]
---
.left-column[
  ## Playbooks
  ## YAML
  ## Obtendo ajuda sobre os módulos
  ## Handlers
  ## Facts
]
.right-column[
Por padrão, a primeira etapa na execução de um _playbook_ é o _Gathering Facts_, a menos que seja desativado com a opção: `gather_facts: no`.
Esta task é a execução do módulo _setup_, este módulo coleta vários valores e preenche na variável _ansible_facts_

- Desativando facts:

```
~$ cat facts.yaml
---
- hosts: testing
  gather_facts: no
  tasks:
    - name:
      debug:
        msg: "ola mundo"
```
]
---
.left-column[
  ## Playbooks
  ## YAML
  ## Obtendo ajuda sobre os módulos
  ## Handlers
  ## Facts
]
.right-column[
- Playbook utilizando os valores de facts:

```
~$ cat facts.yaml
---
- hosts: testing
  tasks:
  - name: IPs
    debug:
      msg: "IP do host são: {{ ansible_all_ipv4_addresses }}"

  - name: O primeiro ip
    debug:
      msg: "O primeiro ip é: {{ ansible_all_ipv4_addresses.0 }}"
...
```

utilizando o `setup`:
```
~$ ansible testing -i inventory -m setup -a filter="ansible_all_ipv4_addresses"
testing | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "172.17.0.2"
        ],
        "discovered_interpreter_python": "/usr/bin/python3.5"
    },
    "changed": false
}
```
]
---
.left-column[
  ## Playbooks
  ## YAML
  ## Obtendo ajuda sobre os módulos
  ## Handlers
  ## Facts
]
.right-column[
- Resultado:

```
~$ ansible-playbook -i inventory fact.yaml 


PLAY [testing] ******************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************
ok: [testing]

TASK [IPs] **********************************************************************************************************
ok: [testing] => {
    "msg": "IP do host são: ['172.17.0.2']"
}

TASK [O primeiro ip] ************************************************************************************************
ok: [testing] => {
    "msg": "O primeiro ip é: 172.17.0.2"
}

PLAY RECAP **********************************************************************************************************
testing                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
]
---
.left-column[
  ## Playbooks
  ## YAML
  ## Obtendo ajuda sobre os módulos
  ## Handlers
  ## Facts
  ## Roles
]
.right-column[
## Roles

É uma forma de automaticamente carregar seus arquivos de variáveis, tasks e handlers baseados numa estrutura de diretórios.

### Estrututura de diretório de uma Role
```
role/
  tasks/
  handlers/
  files/
  templates/
  vars/
  defaults/
  meta/
```
Alguns diretórios dentro da role irão conter um arquivo `main.yml` que deve conter as instruções de cada seção.

]
---
.left-column[
  ## Playbooks
  ## YAML
  ## Obtendo ajuda sobre os módulos
  ## Handlers
  ## Facts
  ## Roles
]
.right-column[
###tasks
Contém a lista das _tasks_ a serem executadas nesta _role_.
###handlers
Contém _handlers_, que podem ser executados por esse _role_ ou qualquer outro fora deste.
###files
Contém os arquivos que pode ser adicionados por esse _role_.
###templates
Contém modelos para serem implantados por esse _role_.
]
---
.left-column[
  ## Playbooks
  ## YAML
  ## Obtendo ajuda sobre os módulos
  ## Handlers
  ## Facts
  ## Roles
]
.right-column[
###vars
Variáveis para uso do _role_.
###defaults
Valores padrões para variáveis deste _role_.
###meta
Define meta dados deste _role_.
]
---
.left-column[
  ## Playbooks
  ## YAML
  ## Obtendo ajuda sobre os módulos
  ## Handlers
  ## Facts
  ## Roles
]
.right-column[
_`roles/exemplo/tasks/main.yml`_
```
- name: Inclui tarefas para o redhat
  import_tasks: redhat.yml
  when: ansible_facts['os_family']|lower == 'redhat'
- name: Inclui tarefas para o debian
  import_tasks: debian.yml
  when: ansible_facts['os_family']|lower == 'debian'
...
```

_`roles/exemplo/tasks/redhat.yml`_
```
---
- yum:
    name: httpd
    state: present
...
```

_`roles/exemplo/tasks/debian.yml`_
```
---
- apt:
    name: apache2
    state: present
...
```
.footnote[Baseado em [exemplo](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html#role-directory-structure) na página do ansbile.]
]
---
.left-column[
  ## Playbooks
  ## YAML
  ## Obtendo ajuda sobre os módulos
  ## Handlers
  ## Facts
  ## Roles
  ## Galaxy
]
.right-column[
**Galaxy** é um [site](https://galaxy.ansible.com/) para compartilhar e encontrar conteúdo de **Ansible**
Possui um cliente, que te permite:
- Criar uma estrutura modelo de role
- Autenticar no site
- Enviar seus roles
- Pesquisar e baixar roles
- etc.
]
---
.left-column[
  ## Playbooks
  ## YAML
  ## Obtendo ajuda sobre os módulos
  ## Handlers
  ## Facts
  ## Roles
  ## Galaxy
]
.right-column[
### Iniciando um role com galaxy

```
~$ ansible-galaxy init teste
- Role teste was created successfully
~$ tree teste/
teste/
├── README.md
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── tasks
│   └── main.yml
├── templates
├── tests
│   ├── inventory
│   └── test.yml
└── vars
    └── main.yml

8 directories, 8 files

```
]
---
layout: inicial
class: center, middle
#Perguntas ?