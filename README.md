import os
import pickle
import bge

# Caminho do arquivo onde os dados serão salvos
arquivo_dados = os.path.join(bge.logic.expandPath('//'), 'dados.pkl')

# Variável global para armazenar os dados dos baús
dados_baus = {}

def adicionar_valores():
    global dados_baus # Usando a variável global
    scene = bge.logic.getCurrentScene()
    cont = bge.logic.getCurrentController()
    obj = cont.owner

    # Criando um dicionário de tuplas com os dados do inventário
    dados = {
        "x1": (550, 55),
        "x2": (2000, 110),
        "x3": (500, 300),
        "x4": (100, 60),
    }

    for i in range(1, 21):
        sensor_name = "x" + str(i).lower()
        
        if sensor_name in cont.sensors and cont.sensors[sensor_name].positive:
            valor_adicionado = dados[sensor_name][1] # Criando uma variável auxiliar
            for j in range(1, 51):
                bau = scene.objects.get("bau" + str(j).lower())
                if bau and bau["quantia"] < dados[sensor_name][0]: # Usando o limite do dicionário
                    bau["item"] = i
                    valor_a_adicionar = max(0, min(valor_adicionado, dados[sensor_name][0] - bau["quantia"])) # Usando a variável auxiliar
                    bau["quantia"] += valor_a_adicionar
                    valor_adicionado -= valor_a_adicionar # Atualizando a variável auxiliar com a variável correta
                    if valor_adicionado == 0: # Se não há mais valor para adicionar, sai do loop
                        break

    todos_cheios = all(bau["quantia"] >= dados.get("x" + str(int(bau.name.replace("bau", ""))), (0, 0))[0] for bau in scene.objects if "bau" in bau.name)

    if todos_cheios:
        bge.logic.sendMessage("cheio")

def remover_valores():
    global dados_baus # Usando a variável global
    scene = bge.logic.getCurrentScene()
    cont = bge.logic.getCurrentController()
    obj = cont.owner

    valores_removidos = {
        "d1": 10,
        "d2": 1,
        "d3": 1,
        "d4": 10,
        "d5": 10,
        "d6": 15,
        "d7": 10,
        "d8": 10,
        "d9": 10,
        "d10": 10,
        "d11": 10,
        "d12": 10,
        "d13": 10,
        "d14": 10,
        "d15": 10,
        "d16": 10,
        "d17": 10,
        "d18": 10,
        "d19": 10,
        "d20": 10,
    }

    for i in range(1, 21):
        sensor_name = "d" + str(i).lower()
        
        if sensor_name in cont.sensors and cont.sensors[sensor_name].positive:
            for j in range(50, 0, -1): # Invertendo a ordem do loop para começar do último baú
                bau = scene.objects.get("bau" + str(j).lower())
                if bau and int(bau["item"]) == i:
                    break
            
            if bau:
                valor_a_remover = max(0, min(valores_removidos[sensor_name], bau["quantia"]))
                bau["quantia"] -= valor_a_remover
                valores_removidos[sensor_name] -= valor_a_remover

                if bau["quantia"] == 0:
                    bau["item"] = 0

def salvar_propriedades():
    global dados_baus # Usando a variável global
    scene = bge.logic.getCurrentScene()

    # Iterando sobre cada baú na cena
    for i in range(1, 51):
        bau = scene.objects.get("bau" + str(i).lower())
        if bau:
            # Salvando as propriedades 'item' e 'quantia' no dicionário
            dados_baus[bau.name] = {'item': bau['item'], 'quantia': bau['quantia']}

    # Salvando os dados no arquivo
    with open(arquivo_dados, 'wb') as f:
        pickle.dump(dados_baus, f)

def carregar_propriedades():
    global dados_baus # Usando a variável global
    scene = bge.logic.getCurrentScene()

    # Carregando os dados do arquivo
    with open(arquivo_dados, 'rb') as f:
        dados_baus = pickle.load(f)

    for i in range(1, 51):
        bau = scene.objects.get("bau" + str(i).lower())
        if bau and bau.name in dados_baus:
            # Carregando as propriedades 'item' e 'quantia' da variável global
            bau['item'] = dados_baus[bau.name]['item']
            bau['quantia'] = dados_baus[bau.name]['quantia']

# Chamando as funções
adicionar_valores()
remover_valores()
salvar_propriedades()
carregar_propriedades()
