name: initial
layout: true
class: center, middle
---
#Ansible
no dia a dia
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

.footnote[contatos: github[/tatsuryu](https://github.com/tatsuryu), telegram: [IcaroTatsu](https://t.me/IcaroTatsu)]
]
---
.left-column[
  ## Quem sou eu?
  ## Contexto
]
.right-column[
### Problemas ao gerir muitos servidores:
<br><br>
- sem padronização nas instalações

- documentação defasada, incompleta ou ausente

- muitos _shellscripts_ que sempre aumentam pra prever todos os casos

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

```
~$ cat nada.yaml
---
name: Teste
type: exemplo
data: |
  alguns valores em
  multiplas linhas.
lista:
  - item 1
  - item 2
  - item 3
content: >
  isto é um multi-line
  formatting
...
```
]
---
.left-column[
  ## Playbooks
  ## YAML
]
.right-column[

YAML parseado:

```
~$ python -c 'import json; import yaml; x=yaml.safe_load(open("nada.yaml","r").read()); print(json.dumps(x, indent=2))'
{
  "name": "Teste",
  "type": "exemplo",
  "data": "alguns valores em\nmultiplas linhas.\n",
  "lista": [
    "item 1",
    "item 2",
    "item 3"
  ],
  "content": "isto \u00e9 um multi-line formatting\n"
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