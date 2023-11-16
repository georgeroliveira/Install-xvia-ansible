# X-Via instruções de instalação

## 1. Instalando o Ansible

Os playbooks requerem o Ansible 2.4 ou posterior. É altamente recomendável o uso da versão estável mais recente do Ansible.

Instale o Ansible seguindo as instruções em [http://docs.ansible.com/ansible/intro_installation.html](http://docs.ansible.com/ansible/intro_installation.html)

## 2. Configuração

#### Inventário

Os hosts para servidores X-Via devem estar listados no inventário Ansible usado (arquivo host).
É recomendável criar seu próprio arquivo host para facilitar melhor a configuração adicional de suas instalações.

O diretório `hosts` contém a lista e as definiões dos servidores a serem utilizados.

Os hosts no arquivo devem estar corretos e completos, porque são usados
na geração de certificados X-Via e definindos durante a instalação. Não use endereços IP.

É possível determinar quais servidores são inicializados preenchendo os grupos
`cs-servers`, `ss-servers`, `cp-servers` e `ca-servers`. Se você não tiver uso para um grupo de servidores específico,
você pode deixar esse grupo vazio.

** Nota: ** Estude a estrutura dos arquivos host de exemplo com cuidado e modele as hierarquias de grupos que você deseja implementar por conta própria
arquivos de inventário. Por exemplo, o grupo `[centos-ss]` para servidores seguros baseados em CentOS LXD-containers é um grupo filho do grupo `[ss-servers]` e pode ser omitido totalmente se você não tiver uso para contâineres CentOS
.
#### Repositórios

O playbook `xvia_dev.yml`, `xvia_qa.yml`, `xvia_prod.yml` utilizam repositórios remotos para instalar o X-Via.
As configurações padrão do repositório são:

* Ubuntu 18 DEB-packages `https://artifactory.xvia.com.br/artifactory/xvia-release/xvia-ubuntu18.04-v0.29.0.tar.gz`
* RHEL-packages `https://artifactory.xvia.com.br/artifactory/xvia-release/xvia-redhat-v0.29.0.tar.gz`.

#### Remote database

A partir do X-Via 6.22.0 é possível configurar o servidor seguro para utilizar um banco de dados remoto. Para fazer isso com o Ansible, é necessário editar os valores das propriedades no arquivo `vars_files/ss_database`.

- `database_host` - URL ou servidor de banco de dados, incluindo a porta ex.: `127.0.0.1:5432`. Defina a propriedade `database_admin_password` se estiver utilizando um banco de dados remoto. Ao usar o banco de dados local, deixe o campo vazio.
- `database_admin_password` - Senha do usuário `postgres`. Ao usar o banco de dados remoto, esse valor precisa ser definido. Caso contrário, deixe em branco.
- `mask_local_postgresql` - Ao usar o banco de dados remoto, geralmente é possível mascarar o banco de dados PostgreSQL local, para que não seja executado em vão. No entanto, em alguns casos extremos, isso não é necessário e essa variável pode ser configurada como false.

As outras propriedades em `vars_files/ss_database` determina os usuários e senhas que o X-Via usa nas conexões.

#### Variáveis adicionais

Parâmetros como nome de usuário e senha do administrador ou valores de DN da Autoridade Certificadora de teste
pode ser configurado usando arquivos no diretório `group_vars`.

No exemplo disponibilzado em [section 3](#3-instalando-o-x-via-usando-ansible) o arquivo de configuração que corresponde ao super grupo `example` é `group_vars/example.yml`.

O arquivo de inventário `example_xroad_hosts.txt` define um grupo de hosts

```
[example:children]
 cs-servers
 ss-servers
 cp-servers
 ca-server
```

contendo todos os outros grupos de hosts do servidor X-Via no inventário. Ansible procura por variáveis em `group_vars/example.yml`
ao inicializar hosts do grupo `example`.

Nome de usuário e senha também são definidos nao diretório `default_vars` da regra `roles/xroad-base`. Esta definição será usada
se nenhum `group_vars` ou outras configurações de precedência mais alta para as mesmas variáveis foram atribuídas.

## 3. Instalando o X-Via usando Ansible

Antes de executar qualquer playbook, certifique-se de configurar seu arquivo de inventário para especificar a composição dos hosts para a instalação do X-Via. Verifique também a sessão [configuration section](#2-configurao) para configurações adicionais.

The following command installs or updates **all X-Via related packages** to latest versions **from package repositories** for all hosts defined in the inventory file `example_xroad_hosts.txt`:

O comando a seguir instala ou atualiza **todos os pacotes relacionados ao X-Via** para todos os hosts definidos no arquivo de inventário `host/dev/xvia`:

```
ansible-playbook -i host/dev/xvia xvia_dev.yml
```