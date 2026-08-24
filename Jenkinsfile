pipeline {
    agent any

    environment {
        // Nome do serviço baseado no seu docker-compose.yml
        SERVICE_NAME = 'api' 
        
        // Caminho do projeto no seu servidor local (Homologação)
        HOMOL_PATH = '/home/bessaz/meu-site-homol'
        
        // Dados do Hostinger (Produção)
        HOSTINGER_USER = 'seu_usuario'
        HOSTINGER_IP   = 'ip.do.hostinger'
        PROD_PATH      = '/caminho/do/projeto/no/hostinger'
    }

    stages {
        stage('🧪 Testes Automatizados') {
            // Este estágio roda em TODAS as branches (main e homol) para garantir a qualidade
            steps {
                echo "Rodando a suíte de testes (Unitários e Integração)..."
                // Exemplo prático usando a estrutura do seu projeto[cite: 1]:
                // sh 'pip install -r requirements-dev.txt'
                // sh 'pytest tests/' 
            }
        }

        stage('🚀 Deploy: Homologação (Servidor Local)') {
            // Só executa se o commit vier da branch 'homol'
            when {
                branch 'homol'
            }
            steps {
                echo "Atualizando ambiente de Homologação local..."
                dir("${HOMOL_PATH}") {
                    // Como o Jenkins está na mesma máquina, executamos o Compose diretamente
                    sh """
                        git pull origin homol
                        docker compose up -d --build
                       
                    """
                }
            }
        }

        stage('👑 Deploy: Produção (Hostinger)') {
            // Só executa se o commit vier da branch 'main'
            when {
                branch 'main'
            }
            steps {
                echo "Conectando ao Hostinger via SSH para atualizar a Produção..."
                // Usa o plugin SSH Agent para conectar no Hostinger sem expor a senha
                sshagent(credentials: ['hostinger-ssh-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${HOSTINGER_USER}@${HOSTINGER_IP} '
                            cd ${PROD_PATH} && \\
                            git pull origin main && \\
                            docker compose -f docker-compose.yml up -d --build && \\
                            docker compose exec -T api alembic upgrade head
                        '
                    """
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finalizado para a branch: ${env.BRANCH_NAME}"
        }
    }
}
