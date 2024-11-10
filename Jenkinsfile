pipeline {
    agent any

    stages {
        stage('Run node-exporter') {
            steps {
                script {
                    // Завантажуємо образ та запускаємо контейнер node-exporter
                    sh '''
                    docker pull sasha22mk/node-exporter:latest
                    docker run -d --name node-exporter -p 9200:9100 \
                    -v /proc:/host/proc:ro -v /sys:/host/sys:ro -v /:/rootfs:ro \
                    sasha22mk/node-exporter:latest \
                    --path.procfs=/host/proc --path.sysfs=/host/sys \
                    --collector.filesystem.mount-points-exclude='^/(sys|proc|dev|host|etc|rootfs/var/lib/docker/containers|rootfs/var/lib/docker/overlay2|rootfs/run/docker/netns|rootfs/var/lib/docker/aufs)($$|/)'
                    '''
                }
            }
        }

        stage('Run Grafana') {
            steps {
                script {
                    // Завантажуємо образ та запускаємо контейнер Grafana
                    sh '''
                    docker volume create grafana-data
                    docker volume create grafana-configs
                    docker pull sasha22mk/grafana:10.1.5-ubuntu
                    docker run -d --name grafana -p 3000:3000 \
                    -v grafana-data:/var/lib/grafana \
                    -v grafana-configs:/etc/grafana \
                    sasha22mk/grafana:10.1.5-ubuntu
                    '''
                }
            }
        }

        stage('Run Prometheus') {
            steps {
                script {
                    // Завантажуємо образ та запускаємо контейнер Prometheus
                    sh '''
                    docker volume create prom-data
                    docker volume create prom-configs
                    docker pull sasha22mk/prometheus:latest
                    docker run -d --name prometheus -p 9090:9090 \
                    -v prom-data:/prometheus \
                    -v prom-configs:/etc/prometheus \
                    sasha22mk/prometheus:latest
                    '''
                }
            }
        }

        stage('Configure Prometheus') {
            steps {
                script {
                    // Оновлюємо конфігурацію prometheus.yml і перезапускаємо Prometheus
                    sh '''
                    cat <<EOF > /var/snap/docker/common/var-lib-docker/volumes/prom-configs/_data/prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
EOF
                    '''
                    
                    sh 'docker stop prometheus'
                    sh 'docker rm prometheus'
                    sh '''
                    docker run -d --name prometheus -p 9090:9090 \
                    -v prom-data:/prometheus \
                    -v prom-configs:/etc/prometheus \
                    sasha22mk/prometheus:latest
                    '''
                }
            }
        }
    }
}
