# Função para abrir um navegador Chrome e retornar a instância do navegador
def abrir_navegador(url):
    service = Service(ChromeDriverManager().install())  # Configura o serviço do driver Chrome
    navegador = webdriver.Chrome(service=service)  # Inicializa o navegador Chrome
    navegador.get(url)  # Abre a URL fornecida no navegador
    return navegador

# Função para esperar e trocar para um iframe específico
def obter_iframe(navegador, iframe_nome):
    wait = WebDriverWait(navegador, 15)
    return wait.until(EC.frame_to_be_available_and_switch_to_it(iframe_nome))

# Função para inserir uma data em um elemento de entrada
def inserir_data(elemento_data, data):
    elemento_data.clear()  # Limpa o elemento de entrada
    elemento_data.send_keys(data)  # Insere a data no elemento de entrada

# Função para clicar em um botão usando XPath
def clicar_botao(navegador, xpath):
    try:
        elemento = WebDriverWait(navegador, 15).until(EC.visibility_of_element_located((By.XPATH, xpath)))
        elemento.click()  # Clica no elemento encontrado
    except TimeoutException as e:
        print(f"TimeoutException: {e}")
        print(f"Elemento XPath '{xpath}' não encontrado dentro do tempo especificado.")

# Função para extrair e salvar o conteúdo de uma tabela
def extrair_conteudo_tabela(navegador, xpath_tabela, arquivo_saida):
    try:
        tabela = WebDriverWait(navegador, 10).until(EC.presence_of_element_located((By.XPATH, xpath_tabela)))
        conteudo_tabela = tabela.text

        if conteudo_tabela.strip():
            conteudo_tabela_formatado = ",".join(conteudo_tabela.split())

            print(f"Conteúdo da tabela {xpath_tabela}:\n{conteudo_tabela_formatado}")

            with open(arquivo_saida, 'a', encoding='utf-8') as arquivo:
                arquivo.write(f"{conteudo_tabela_formatado}\n\n")
        else:
            print(f"Conteúdo da tabela {xpath_tabela} está vazio. Não será salvo.")

    except TimeoutException as e:
        print(f"TimeoutException: {e}")
        print(f"Tabela XPath '{xpath_tabela}' não encontrada dentro do tempo especificado.")

# Função para esperar a presença de um elemento
def esperar_elemento_presente(driver, by, value, timeout=15):
    try:
        return WebDriverWait(driver, timeout).until(EC.presence_of_element_located((by, value)))
    except TimeoutException:
        print(f"Timeout esperando pelo elemento {by}: {value}")
        raise

# Função para esperar que um elemento seja clicável
def esperar_elemento_clicavel(driver, by, value, timeout=15):
    try:
        return WebDriverWait(driver, timeout).until(EC.element_to_be_clickable((by, value)))
    except TimeoutException:
        print(f"Timeout esperando pelo elemento clicável {by}: {value}")
        raise

# Função principal
def main():
    url = 'https://www.weather.gov/wrh/Climate?wfo=mtr'
    navegador = None

    try:
        navegador = abrir_navegador(url)

        iframe_nome = "routine"
        obter_iframe(navegador, iframe_nome)

        for month in range(1, 13):
            month_str = f"{month:02d}"
            data = f"2000-{month_str}"

            elemento_data = esperar_elemento_presente(navegador, By.XPATH, '//*[@id="tDatepicker"]')
            inserir_data(elemento_data, data)
            clicar_botao(navegador, "//*[@id='go']")

            arquivo_saida = 'conteudo_tabelas.txt'
            
            # Gera os xpaths das tabelas dinamicamente verificando se o elemento está presente
            xpaths_tabelas = []
            # Itera de 1 a 30 para criar os xpaths das tabelas usando um padrão específico
            for i in range(1, 31):
                xpath_tabela = f'//*[@id="results_area"]/table[1]/tbody/tr[{i}]'
                # Verifica se o elemento da tabela está presente na página
                if navegador.find_elements(By.XPATH, xpath_tabela):
                    xpaths_tabelas.append(xpath_tabela)

            # Aguarda a presença de cada tabela na página antes de prosseguir
            for xpath_tabela in xpaths_tabelas:
                esperar_elemento_presente(navegador, By.XPATH, xpath_tabela)

            # Extrai o conteúdo de cada tabela usando os xpaths gerados anteriormente
            for xpath_tabela in xpaths_tabelas:
                extrair_conteudo_tabela(navegador, xpath_tabela, arquivo_saida)
                time.sleep(3)  # Espera de 3 segundos

            # Aguarda a presença do botão de fechar janela antes de clicar
            esperar_elemento_presente(navegador, By.XPATH, "/html/body/div[5]/div[1]/button[3]/span[1]")
            time.sleep(3)  # Espera de 3 segundos
            # Obtém o elemento clicável para fechar a janela
            fechar_janela_botao = esperar_elemento_clicavel(navegador, By.XPATH, "/html/body/div[5]/div[1]/button[3]/span[1]")
            time.sleep(3)  # Espera de 3 segundos
            # Clica no botão para fechar a janela
            clicar_botao(navegador, "/html/body/div[5]/div[1]/button[3]/span[1]")
            time.sleep(3)  # Espera de 3 segundos

    except Exception as e:
        print(f"Erro: {e}")
    finally:
        if navegador:
            try:
                navegador.quit()
            except Exception as e:
                print(f"Erro ao fechar o navegador: {type(e).__name__}: {e}")

if __name__ == "__main__":
    try:
        main()
    except Exception as e:
        print(f"Erro em main(): {type(e).__name__}: {e}")
