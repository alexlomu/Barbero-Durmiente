# Barbero-Durmiente

El link del repositorio es el siguiente: [GitHub](https://github.com/alexlomu/Barbero-Durmiente)

## Resolución del problema

El código propueto es el siguiente:

    import threading
    import time
    import random

    MAX_CLIENTES = 15  # número máximo de clientes que pueden llegar
    MAX_ESPERA = 5  # tiempo máximo de espera en segundos antes de irse

    clientes_esperando = 0  # contador de clientes esperando
    mutex = threading.Semaphore(1)  # semáforo para proteger la sección crítica
    cliente = threading.Semaphore(0)  # semáforo para despertar al barbero
    silla = threading.Semaphore(0)  # semáforo para indicar que hay un cliente en la silla del barbero
    corte_terminado = threading.Semaphore(0)  # semáforo para indicar que el corte de pelo ha terminado


    def cortar_pelo():
        print("El barbero está cortando el pelo...")
        tiempo = round(random.uniform(1, 2), 2)
        time.sleep(tiempo)
        print("El corte de pelo ha terminado en", tiempo, "segundos")
        corte_terminado.release()  # indica que el corte de pelo ha terminado


    def esperar():
        global clientes_esperando
        print("Cliente esperando...")
        tiempo = round(random.uniform(1, MAX_ESPERA), 2)
        time.sleep(tiempo)
        if clientes_esperando > 0:  # si hay alguien esperando
            mutex.acquire()  # protege la sección crítica
            clientes_esperando -= 1  # decrementa el contador de clientes esperando
            mutex.release()  # libera la sección crítica
            cliente.release()  # despierta al barbero
            silla.acquire()  # espera a que el barbero indique que la silla está libre
            print("El cliente está recibiendo un corte de pelo...")
            corte_terminado.acquire()  # espera a que el corte de pelo termine
            silla.release()  # indica que la silla está libre
        else:  # si no hay nadie esperando
            print("El cliente se fue después de esperar", tiempo, "segundos")


    def barbero():
        while True:
            print("El barbero está durmiendo...")
            cliente.acquire()  # espera a que llegue un cliente
            silla.release()  # indica que la silla está ocupada
            cortar_pelo()  # corta el pelo del cliente


    if __name__ == "__main__":
        # crea el hilo del barbero
        barbero_thread = threading.Thread(target=barbero, daemon=True)
        barbero_thread.start()

        # crea los hilos de los clientes
        for i in range(MAX_CLIENTES):
            cliente_thread = threading.Thread(target=esperar)
            cliente_thread.start()
            mutex.acquire()  # protege la sección crítica
            clientes_esperando += 1  # incrementa el contador de clientes esperando
            mutex.release()  # libera la sección crítica
            time.sleep(0.5)  # espera un poco antes de crear otro cliente
           
           
           
           
           
El output recibido es el siguiente:
    
    El barbero está durmiendo...
    Cliente esperando...
    Cliente esperando...
    Cliente esperando...
    Cliente esperando...
    Cliente esperando...
    El barbero está cortando el pelo...El cliente está recibiendo un corte de pelo...

    Cliente esperando...
    Cliente esperando...
    El corte de pelo ha terminado en 1.19 segundos
    El barbero está durmiendo...El cliente está recibiendo un corte de pelo...

    El barbero está cortando el pelo...
    Cliente esperando...
    El cliente está recibiendo un corte de pelo...
    Cliente esperando...
    Cliente esperando...
    El corte de pelo ha terminado en 1.11 segundos
    El barbero está durmiendo...
    .
    .
    .
