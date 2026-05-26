import cv2

import numpy as np

import mss

import win32api, win32con

import time



def iniciar_auxilio_mira 🎯 precisao=0.40, proximidade=100):

    tempo_por_frame = 1.0 / _🎯 

    

    # Configuração da cor de detecção (Vermelho HSV)

    cor_minima = np.array([0, 120, 70])

    cor_maxima = np.array([10, 255, 255])



    # Instancia o capturador ultra-rápido

    sct = mss.mss()

    

    # Pega a resolução do monitor principal

    monitor = sct.monitors[1]

    tela_largura = monitor["width"]

    tela_altura = monitor["height"]

    

    # Define a zona de captura centralizada

    caixa_busca = {

        "top": int(tela_altura // 2 - proximidade),

        "left": int(tela_largura // 2 - proximidade),

        "width": int(proximidade * 2),

        "height": int(proximidade * 2)

    }



    def mover_mouse(x, y):

        # Mover de forma relativa

        win32api.mouse_event(win32con.MOUSEEVENTF_MOVE, int(x), int(y), 0, 0)



    print(f"[!] Script Pronto | FPS Alvo: {fps_alvo} | Suavização: {precisao*100}%")

    print("[!] SEGURE O BOTÃO DIREITO DO MOUSE (M2) para ativar.")

    

    # Centralizado dentro da própria caixa de busca

    centro_caixa_x = proximidade

    centro_caixa_y = proximidade



    while True:

        tempo_inicial = time.perf_counter()



        # Verifica se o Botão Direito (0x02) está pressionado

        if win32api.GetAsyncKeyState(0x02) < 0:

            

            # Captura de tela ultra rápida com mss

            print_tela = sct.grab(caixa_busca)

            frame = np.array(print_tela)

            

            # MSS captura em BGRA, convertemos para HSV

            hsv = cv2.cvtColor(frame, cv2.COLOR_BGRA2BGR)

            hsv = cv2.cvtColor(hsv, cv2.COLOR_BGR2HSV)



            mascara = cv2.inRange(hsv, cor_minima, cor_maxima)

            contornos, _ = cv2.findContours(mascara, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

            

            if contornos:

                maior_contorno = max(contornos, key=cv2.contourArea)

                

                if cv2.contourArea(maior_contorno) > 80: # Ajustado para evitar falsos positivos

                    M = cv2.moments(maior_contorno)

                    

                    if M["m00"] != 0:

                        alvo_x = int(M["m10"] / M["m00"])

                        alvo_y = int(M["m01"] / M["m00"])

                        

                        # Distância do centro da caixa até o centro do inimigo

                        distancia_x = alvo_x - centro_caixa_x

                        distancia_y = alvo_y - centro_caixa_y

                        

                        # Multiplica pela precisão/suavização (ex: move só 40% do caminho por frame)

                        movimento_x = distancia_x * precisao

                        movimento_y = distancia_y * precisao

                        

                        mover_mouse(movimento_x, movimento_y)



        # Pequena folga para o processador se não atingir o tempo

        tempo_decorrido = time.perf_counter() - tempo_inicial

        tempo_restante = tempo_por_frame - tempo_decorrido

        if tempo_restante > 0:

            time.sleep(tempo_restante)



if __name__ == "__main__":

    # Para instalar as dependências que faltam: pip install mss

    iniciar_auxilio_mira(fps_alvo=60, precisao=0.40, proximidade=120)[![Build Status](https://github.com/dart-lang/pub/workflows/Dart%20CI/badge.svg)](https://github.com/dart-lang/pub/actions/workflows/test.yaml?query=workflow%3A%22Dart+CI%22+branch%3Amaster)

Pub is the package manager for Dart.

# Contributing to pub

Thanks for being interested in contributing to pub! Contributing to a new
project can be hard: there's a lot of new code and practices to learn. This
document is intended to get you up and running as quickly as possible. For
more information, see the
[pub tool documentation](https://dart.dev/tools/pub/cmd).

The first step towards contributing is to contact the pub dev team and let us
know what you're working on, so we can be sure not to start working on the same
thing at the same time. Open an [issue](http://github.com/dart-lang/pub/issues/new)
letting us know that you're interested in contributing and what you plan on working on.
This will also let us give you specific advice about where to start.

## Organization

Pub isn't a package, but it's organized like one. It has four top-level
directories:

* `lib/` contains the implementation of pub. Currently, it's all in `lib/src/`,
  since there are no libraries intended for public consumption.

* `test/` contains the tests for pub.

* `bin/` contains `pub.dart`, the entrypoint script that's run whenever a user
  types "pub" on the command line or runs it in the Dart editor. This is usually
  run through shell scripts in `sdk/bin` at the root of the Dart repository.

It's probably easiest to start diving into the codebase by looking at a
particular pub command. Each command is encapsulated in files in
`lib/src/command/`.

## Running pub

To run pub from the Git repository, run:

    dart bin/pub.dart

## Testing pub

Before any change is made to pub, all tests should pass. To run a pub test, run:

    dart tool/test.dart test/path/to_test.dart

To run all tests at once, run:

    dart tool/test.dart

Changes to pub should be accompanied by one or more tests that exercise the new
functionality. When adding a test, the best strategy is to find a similar test
in `test/` and follow the same patterns.

Pub tests come in two basic forms. The first, which is usually used to unit test
classes and libraries internal to pub, has many tests in a single file. This is
used when each test will take a short time to run. For example,
`test/version_test.dart` contains unit tests for pub's Version class.

The other form, used by most pub tests, is usually used for integration tests of
user-visible pub commands. Each test has a file to itself, which is named after
the test description. This is used when tests can take a long time to run to
avoid having the tests time out when running on the build bots. For example,
`tests/get/hosted/get_transitive_test.dart` tests the resolution of transitive
hosted dependencies when using `dart pub get`/`flutter pub get`.

## Landing your patch

All patches to official Dart packages, including to pub, need to undergo code
review before they're submitted. The full process for putting up your patch for
review is [documented elsewhere][contributing].

[contributing]: https://github.com/dart-lang/sdk/blob/main/CONTRIBUTING.md
