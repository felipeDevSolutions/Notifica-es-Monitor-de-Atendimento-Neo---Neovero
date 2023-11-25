# Código do sistema
    from playwright.sync_api import sync_playwright
    import time
    import os
    from pathlib import Path
    import pygame

    os.environ["PLAYWRIGHT_BROWSERS_PATH"] = os.path.join(os.getcwd(), "chromium")

    # Obtém o diretório atual onde o arquivo "Notificar.py" está localizado
    current_directory = os.path.dirname(os.path.abspath(__file__))

    # Define o caminho para a pasta com os áudios
    audio_folder_path = os.path.join(current_directory, 'static')

    def main():

        pygame.mixer.init()

        # Obtém o local do audio para Requisições para Equipe Técnica
        audioAreaTecnica = os.path.join('static', 'Nova requisicao para area tecnica.mp3')
        audio_pathAreaTecnica = Path(audioAreaTecnica).resolve().as_posix()

        # Obtém o local do audio para Requisições para Equipe de Logística
        audioLogistica = os.path.join('static', 'Nova requisicao para logistica.mp3')
        audio_pathLogistica = Path(audioLogistica).resolve().as_posix()

        # Obtém o local do audio para Requisições da Unimed Lar
        audioUnimedLar = os.path.join('static', 'Nova requisicao na base da Unimed Lar.mp3')
        audio_pathUnimedLar = Path(audioUnimedLar).resolve().as_posix()

        # Obtém o local do audio para Requisições das Clínicas
        audioClinicas = os.path.join('static', 'Nova requisicao na base das Clinicas.mp3')
        audio_pathClinicas = Path(audioClinicas).resolve().as_posix()

        # Obtém o local do audio para Requisições dos Laboratórios
        audioLaboratorios = os.path.join('static', 'Nova requisicao na base dos laboratorios.mp3')
        audio_pathLaboratorios = Path(audioLaboratorios).resolve().as_posix()

        # Obtém o local do audio para Requisições da Unimed Urgente
        audioUnimedUrgente = os.path.join('static','Nova requisicao na base da Unimed Urgente.mp3')
        audio_pathUnimedUrgente = Path(audioUnimedUrgente).resolve().as_posix()

        # Obtém o local do audio para Requisições da Medicina Preventiva
        audioMedPrev = os.path.join('static', 'Nova requisicao na base da medicina preventiva.mp3')
        audio_pathMedPrev = Path(audioMedPrev).resolve().as_posix()

        #Inicia a quantidade de requisição como none
        requisicao = None

        # Variáveis adicionais para controlar a reprodução do áudio
        reproduzir_audio = False
        ultimo_valor_reproduzido = None

        with sync_playwright() as p:
            browser = p.chromium.launch(headless=False)
            context = browser.new_context()

            # Abre a página
            page = context.new_page()
            page.set_viewport_size({"width": 1366, "height": 768})
            page.goto("https://unimedfortaleza.neovero.com/login")

            # Preenche as credenciais de login
            page.fill('input[type="text"]', "painel.ec")
            page.fill('input[type="password"]', "unimed")
            page.click('button[type="submit"]')

            # Aguarda a página carregar
            page.wait_for_load_state("load")

            page.wait_for_selector('[data-empresaid="1"]', state="visible")
            page.click('[data-empresaid="1"]')

            #Clica em Ordem de Serviço
            page.wait_for_selector('//*[@id="nvNavBar"]/nav[2]/div[1]/ul/li[3]/a', state="visible")
            page.click('//*[@id="nvNavBar"]/nav[2]/div[1]/ul/li[3]/a')
            time.sleep(1)

            #Clica em Monitor de Atendimento Neo
            page.click('//*[@id="nvNavBar"]/nav[2]/div[1]/ul/li[3]/div/ul/li[8]/a[2]')

            #Espera o botão Monitor Corporativo aparecer na tela
            page.wait_for_selector('//*[@id="janela0"]/nav/div[1]/ul/li[2]/div/div[2]', state="visible")

            #Clica no botão Monitor Corporativo
            page.click('//*[@id="janela0"]/nav/div[1]/ul/li[2]/div/div[2]')

            while (True):

                # Aguarda a aparição do elemento da Linha de requisição
                elemento = '.cdk-row.ng-star-inserted:nth-child(1)'
                page.wait_for_selector(elemento, state='visible', timeout=0)

                #Linha de uma nova requisição
                novaReq = page.inner_text(elemento)

                # Apenas o Número da linha de requisição
                numeroReq = page.inner_text(f'{elemento} .cdk-column-numero')

                # Apenas o tipo da requisição
                tipoReq = page.inner_text(f'{elemento} .cdk-column-tipoManutencao')

                # Apenas a Emresa
                empresaReq = page.inner_text(f'{elemento} .cdk-column-empresaid')

                # Atribua ReqLogistica apenas quando o tipo de requisição for "LOGÍSTICA"
                ReqLogistica = tipoReq if page.inner_text(f'{elemento} .cdk-column-tipoManutencao') == "LOGÍSTICA" else None

                # Lógica do loop para reproduzir o áudio da notificação
                if int(numeroReq) != requisicao:
                    requisicao = numeroReq
                    
                    if numeroReq != None and page.is_visible(f'{elemento} .cdk-column-numero'):
                        if ultimo_valor_reproduzido is None or int(numeroReq) > int(ultimo_valor_reproduzido):
                            reproduzir_audio = True
                        else:
                            reproduzir_audio = False
                    else:
                        reproduzir_audio = False

                    #Verificar a empresa e executa um áudio específico para cada empresa 
                    if empresaReq == 'HOSPITAL UNIMED FORTALEZA':

                        if tipoReq == 'LOGÍSTICA':
                            if reproduzir_audio:
                                # Reproduzir o áudio usando o mixer do pygame
                                pygame.mixer.music.load(audio_pathLogistica)
                                pygame.mixer.music.play()
                                ultimo_valor_reproduzido = numeroReq
                        else: 
                            if reproduzir_audio:
                                # Reproduzir o áudio usando o mixer do pygame
                                pygame.mixer.music.load(audio_pathAreaTecnica)
                                pygame.mixer.music.play()
                                ultimo_valor_reproduzido = numeroReq

                    elif empresaReq == 'UNIMED LAR':
                        if reproduzir_audio:
                            # Reproduzir o áudio usando o mixer do pygame
                            pygame.mixer.music.load(audio_pathUnimedLar)
                            pygame.mixer.music.play()
                            ultimo_valor_reproduzido = numeroReq
                    
                    elif empresaReq == 'CLÍNICAS UNIMED':
                        if reproduzir_audio:
                            # Reproduzir o áudio usando o mixer do pygame
                            pygame.mixer.music.load(audio_pathClinicas)
                            pygame.mixer.music.play()
                            ultimo_valor_reproduzido = numeroReq
                    
                    elif empresaReq == 'LABORATÓRIOS UNIMED':
                        if reproduzir_audio:
                            # Reproduzir o áudio usando o mixer do pygame
                            pygame.mixer.music.load(audio_pathLaboratorios)
                            pygame.mixer.music.play()
                            ultimo_valor_reproduzido = numeroReq

                    elif empresaReq == 'UNIMED URGENTE':
                        if reproduzir_audio:
                            # Reproduzir o áudio usando o mixer do pygame
                            pygame.mixer.music.load(audio_pathUnimedUrgente)
                            pygame.mixer.music.play()
                            ultimo_valor_reproduzido = numeroReq

                    elif empresaReq == 'MEDICINA PREVENTIVA UNIMED FORTALEZA':
                        if reproduzir_audio:
                            # Reproduzir o áudio usando o mixer do pygame
                            pygame.mixer.music.load(audio_pathMedPrev)
                            pygame.mixer.music.play()
                            ultimo_valor_reproduzido = numeroReq  

                    elif empresaReq == 'CENTRO ONCOLÓGICO UNIMED FORTALEZA':
                        print("Nova requisição do CENTRO ONCOLÓGICO UNIMED FORTALEZA")

                    else:
                        print("Nova requisição do HOSPITAL UNIMED SUL")

                else:
                    print("Aguardando nova requisição")
            
    if __name__ == "__main__":
        main()

# Como fazer o deploy e gerar um arquivo executável usando o PyInstaller

    Primeiro precisa criar um ambiente virtual e instalar as dependências, nesse caso:
        •	Playwright
        •	Pygame
        •	PyInstaller

        Para criar o ambiente virtual:
            Crie um Ambiente Virtual:
                • Abra um terminal na pasta do seu projeto.
                • Execute o seguinte comando para criar um ambiente virtual:
                     python -m venv venv

            Ative o ambiente virtual:
                • No Windows:
                     .\venv\Scripts\activate

                • Se precisar desbloquear a política de execução no PowerShell use o código:
                    • Abra o PowerShell e digite o código:
                    • Set-ExecutionPolicy RemoteSigned
                    • Ative o ambiente virtual
                    • Bloquei novamente a política de execução de scripts
                    • Set-ExecutionPolicy Restricted

        Instale as Dependências:
            • Com o ambiente virtual ativado, instale as dependências do seu projeto:
                 pip install playwright pygame pyinstaller

        Execute o comando PyInstaller:
            • Após a instalação do PyInstaller, execute o comando de empacotamento:
                 pyinstaller --onefile "Notificar Requisições.py"

# Arquivos necessários para o sistema rodar

    Dentro da pasta Dist (Para funcionar em qualquer computador):
        * Crie uma pasta com o nome chromium/chromium-1084 e cole a pasta do Playwright
        
    Para conseguir executar na sua IDE:
        * Coloque a pasta chromium também dentro da pasta do projeto

			
