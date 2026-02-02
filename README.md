# Monitoramento com Prometheus e Grafana: Um Guia Prático

Se você chegou até aqui, provavelmente quer entender como colocar de pé uma estrutura de monitoramento robusta usando Docker. O coração desse projeto bate em dois arquivos principais: o `docker-compose.yml` e o `prometheus.yml`.

Neste guia, vou te levar pela mão e explicar o que cada linha faz, de um jeito simples e direto.

---

## 1. O Maestro: `docker-compose.yml`

Imagine o Docker Compose como o maestro de uma orquestra. Ele diz quem deve tocar, quando começar e como cada músico deve se comunicar com o outro. 

Aqui está o que está acontecendo dentro dele:

### A Rede (`networks`)
Criamos uma rede chamada `monitoring`. Isso é fundamental para que os nossos containers consigam conversar entre si usando apenas o nome do serviço (como se fosse um DNS interno).

### Os Serviços (Nossos Contêineres)

1.  **Node Exporter**: Ele é o "espião" do sistema operacional. Ele coleta métricas da máquina física (CPU, memória, disco) e as expõe na porta `9100`.
2.  **cAdvisor**: Enquanto o Node Exporter olha para a máquina, o cAdvisor olha para os **containers**. Ele nos diz quanto cada processo Docker está consumindo de recursos. Repare que ele precisa de vários "volumes" (pastas do sistema) montados como `ro` (read-only) para conseguir ler os dados do Docker.
3.  **Prometheus**: É o nosso banco de dados de séries temporais. É ele quem vai "perguntar" as métricas para o Node Exporter e para o cAdvisor a cada poucos segundos. 
    *   **Volume**: Conectamos o nosso arquivo local `prometheus.yml` para dentro do container.
    *   **Retenção**: Configuramos `--storage.tsdb.retention.time=7d` para guardar os dados por apenas 7 dias, evitando que o disco da sua VM encha rápido demais.
4.  **Grafana**: A parte bonita da festa. O Grafana vai se conectar ao Prometheus para criar aqueles gráficos e dashboards incríveis. Ele tem um volume persistente (`grafana-data`) para que você não perca suas configurações se o container for reiniciado.

---

## 2. O Cérebro: `prometheus.yml`

Se o Docker Compose monta a estrutura, o `prometheus.yml` define as regras de negócio: **quem** monitorar e **com que frequência**.

```yaml
global:
  scrape_interval: 30s
```
Aqui definimos que o Prometheus vai buscar novos dados a cada 30 segundos. Escolhemos esse intervalo para ser gentil com máquinas de menor performance (como as instâncias gratuitas da Oracle ou AWS).

### As Configurações de Coleta (`scrape_configs`)

Configuramos dois "trabalhos" (jobs):

*   **Job 'node'**: Diz ao Prometheus para ir até o container `node-exporter` na porta `9100`. Lembra da rede que criamos no Docker Compose? É por causa dela que podemos usar o nome `node-exporter` em vez de um endereço IP.
*   **Job 'cadvisor'**: Mesma lógica, mas aponta para o serviço que monitora os containers na porta `8080`.

---

## 3. Como colocar tudo para rodar?

Com os dois arquivos na mesma pasta, o processo é quase mágico. Basta abrir o terminal e digitar:

```bash
docker-compose up -d
```

O `-d` serve para rodar em modo "detached" (em segundo plano). Depois disso, você terá:

*   **Prometheus**: `http://seu-ip:9090`
*   **Grafana**: `http://seu-ip:3000` (Login padrão: admin/admin)
*   **Métricas do Sistema**: `http://seu-ip:9100/metrics`
*   **Métricas dos Containers**: `http://seu-ip:8080/metrics`

---

## Conclusão

Ter um sistema de monitoramento não precisa ser complexo. Com esses dois arquivos, você transformou uma máquina comum em um servidor monitorado profissionalmente. Agora, o próximo passo é entrar no Grafana, adicionar o Prometheus como *Data Source* e começar a criar seus dashboards! 🚀
