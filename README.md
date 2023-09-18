# apache-playground2-lazy

### Load Balance
Load balancer é a prática de distribuir cargas de trabalho computacional entre dois ou mais computadores. O load balancer reduz a pressão em cada servidor e o torna mais eficiente acelerando o desempenho e reduzindo a latência. Para facilitar o entendimento: imagine um caixa de supermercado atendendo todos os clientes? não é nada prático. O Load Balancer pode funcionar de duas maneiras: Estática ou Dinâmica

Estática: Feito de maneira aleatória, sem ter conhecimento da carga atual. Rápido de configurar, porém pouco eficiente.(Dns Round Robin)

Dinâmico: Nesse caso, é levado em conta diferentes fatores, o que torna a sua configuração um pouco mais complicada. Podem transferir tráfego de servidores sobrecarregados para servidores subutilizados ou com pouca pressão de trabalho. O balanceamento dinâmico pode ocorrer de diversas maneiras, uma delas, por exemplo, é por geolocalização ou recursos dos servidores.

Fail over: Fail over é quando um determinado servidor cai e um balanceador de carga distribui o trabalho para outro grupo de servidores. 

### Load Balance Configuration in Apache

```bash
sudo a2enmod proxy // Habilitação do Proxy
sudo a2enmod proxy_http // Habilitação do proxy http
sudo a2enmod proxy_balancer // Módulo de proxy com balancer
sudo a2enmod lbmethod_byrequests // Módulo responsável por repassar a carga de trabalho para outros workers
sudo service apache2 restart // Necessário reiniciar servidor
```

### Fail Over Configuration in apache
Vamos imaginar o seguinte cenário: O dominio número 1 e dominio 2 só vai enviar tráfico se o dominio x ou dominio y estiveram comprometidos, ou seja, fora do ar. Se todos os dominio estiveram setados para 0 um novo dominio vai entrar em ação. Em um cenário de Fail Over temos que mapear todas as saídas necessárias para caso um servidor esteja fora do ar. Para isso, devemos configurar uma série de balancers e works com a diretiva Proxy Pass. o load factor sugere que o backend irá aguentar 3x de carga a mais com um tempo de 1 segundo

```bash
<Proxy balancer://myset>
    BalancerMember http://www2.example.com:8080
    BalancerMember http://www3.example.com:8080 loadfactor=3 timeout=1
    BalancerMember http://spare1.example.com:8080 status=+R
    BalancerMember http://spare2.example.com:8080 status=+R
    BalancerMember http://hstandby.example.com:8080 status=+H
    BalancerMember http://bkup1.example.com:8080 lbset=1
    BalancerMember http://bkup2.example.com:8080 lbset=1
    ProxySet lbmethod=byrequests
</Proxy>

ProxyPass "/images/"  "balancer://myset/"
ProxyPassReverse "/images/"  "balancer://myset/"
```

### Referências
https://httpd.apache.org/docs/2.4/howto/reverse_proxy.html

https://www.cloudflare.com/en-gb/learning/performance/what-is-load-balancing/

https://httpd.apache.org/docs/2.4/mod/mod_lbmethod_byrequests.html

https://ubiq.co/tech-blog/apache-load-balancer-configuration/
