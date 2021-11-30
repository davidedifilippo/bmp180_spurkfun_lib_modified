## Utilizzo della libreria bmp180

La libreria serve per interrogare il sensore e recuperare il valore di temperatura e pressione atmosferica

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
  
Se la lettura va a buon fine, stampiamo T:

    if (ok)
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

      if (ok){
      Serial.print("Pressione assoluta: ");
      Serial.print(P,2);
      Serial.println(" millibar");
      }

Attendiamo prima di interrogare di nuovo il sensore.

        delay(200);
        
###Pressione relativa        

Il sensore di pressione restituisce la pressione assoluta, che varia con l'altitudine. Per rimuovere gli effetti dell'altitudine, 
si può utilizzare la funzione Questo numero è comunemente usato nei bollettini meteorologici.
    

          p0 = bmp180.seaLevel(P,ALTITUDINE); // siamo a 400 metri 
          
 - P = pressione assoluta in mb 
 - ALTITUDINE = altitudine attuale in metri
 - Risultato: p0 = pressione compensata a livello del mare in mb
          
          Serial.print("pressione relativa (a livello del mare): ");
          Serial.print(p0,2);
          Serial.println("mb");
  
##Altitudine          

Se vuoi determinare l'altitudine dalla lettura della pressione, si può utilizzre la funzione altitudine insieme a una pressione di base (livello del mare o altro).
Parametri: 

          a = bmp180.altitudine(P, p0);

- P = pressione assoluta in mb 
- p0 = pressione di base in mb.
- a = altitudine in m

          
          Serial.print("Altitudine calcolata: ");
          Serial.print(a,0);
          Serial.println("metri");
  
