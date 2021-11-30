## Utilizzo della libreria bmp180

La libreria serve per interrogare il sensore e recuperare il valore di temperatura e pressione atmosferica

### Utilizzo della libreria


Salvare i due file:

- **bmp180.h**
- **bmp180.c**

nella cartella del programma di gestione.  Includere il file **bmp180.h** nel programma principale:

    #include “bmp180.h”   
    
Da questo momento è disponibile la classe di dispositivi bmp180. Si crea quindi un oggetto del tipo MPU6050.

    BMP180 bmp180;
    
### Setup
    
Prima di poter utilizzare il sensore è necessario prelevare dalla memoria **EEPROM** a sola lettura del sensore i parametri di calibrazione. Serviranno **alla libreria** per il calcolo di precisione della temperatura e della pressione:

    bmp180.begin() //da inserire nella fase di setup 

Questa chiamata restituisce "true" se il sensore risponde all'interrogazione. In caso contrario ci potrebbe essere un problema nel collegamento del sensore. Sarebbe buona norma controllare:

    if(bmp180.begin()) Serial.println("ok"); else Serial.println("Inizializzazione fallita);
    
### Loop - fase di monitoraggio ciclico 
    
Per poter acquisire la pressione atmosferica in modo corretto bisogna conoscere la temperatura dell’ambiente in cui si effettua la misura. 

    int attesa = bmp180.startTemperature();

La chiamata precedente attiva il sensore e comanda l'aquisizione della temperatura. Il valore resituito è il **tempo da attendere in millisecondi necessario al sensore per la conversione numerica della temperatura tramite ADC interno** 

    if (attesa != 0) delay(attesa);

Il dato a 16 bit è ora presente all’indirizzo 0xF6 nella memoria interna del sensore. Basta ora prelevare con due letture consecutive l’MSB e LSB del dato. La ricostruzione del dato è difficoltosa in questo caso a causa dei parametri di calibrazione. Ci affidiamo alla libreria:

    double T; //variabile in virgola mobile a doppia precisione in cui verrà memorizzata la temperatura (sarebbe meglio globale)
    
    bool ok = bmp180.getTemperature(T);
  
Se la lettura va a buon fine 

    if (ok != false)
    {
      Serial.print("temperatura: ");
      Serial.print(T,2);             //Stampa del dato con due cifre significative
      Serial.println(" °C");
    }

In modo analogo possiamo rilevare la pressione atmosferica:
      
      status = pressure.startPressure(3); // 3 è il livello di precisione massimo della misura
      if (status != 0) delay(status);
      
      double P;  //sarebbe meglio globale
      bool ok = bmp180.getPressure(P,T);

      if (ok != 0){
      Serial.print("Pressione assoluta: ");
      Serial.print(P,2);
      Serial.println(" millibar");
      }


  
