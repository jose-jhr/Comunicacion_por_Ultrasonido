**Comunicación por ultrasonido.**

Video de fredy vega explicando la complejidad del virus Stuxnet.

<a href="https://youtu.be/gwEq0-ACUr8?si=C6XIkI40u1sTYTNc&t=730">
  <img src="https://static.platzi.com/blog/uploads/2016/09/freddy-vega.jpg" width="450">
</a>



Aunque en las investigaciones que se hicieron en este canal no se encontro que la forma de comunicación y dispersión del virus
fue el ultrasonido, se encontro que si existio un virus llamado **BadBios**

Para dar solución a esta tarea que es emplear el sonido para la comunicación entre dos dispositivos realizaremos las siguientes tareas.

posible error en Android. 

```
implementation 'androidx.appcompat:appcompat:1.5.0'
```

Implementación en buil.gradle, para hacer referencia a la vistas en xml.
```
 buildFeatures{
        viewBinding = true
    }
```

**1) Implementar la libreria tensorflow task audio**

Esta biblioteca permite utilizar modelos de TensorFlow Lite específicamente diseñados para tareas de procesamiento de audio en aplicaciones móviles.
```
  //tensorflow audio task
  implementation 'org.tensorflow:tensorflow-lite-task-audio:0.2.0'
```
**2) Ahora vamos a implementar el modelo generado, puedes usar mi modelo.**

**Mi modelo:** https://cienciayculturacreativa.com/2024/comunicacion_ultrasonido/modelsound10.tflite

**Etiquetas:** https://cienciayculturacreativa.com/2024/comunicacion_ultrasonido/labels.txt

**Proyecto Teachable Machine Ejemplo:** https://cienciayculturacreativa.com/2024/comunicacion_ultrasonido/project.tm


**3) Ahora vamos a implementar la clase que sera encargada de clasificar las señales de audio.**

```kotlin 
import android.content.Context
import android.provider.MediaStore.Audio
import org.tensorflow.lite.task.audio.classifier.AudioClassifier
import org.tensorflow.lite.task.audio.classifier.Classifications
import java.util.Timer
import kotlin.concurrent.scheduleAtFixedRate

interface TensorResultListener {  // Define una interfaz para el listener de resultados del tensor
    fun onTensorResult(results: List<String>)  // Método para recibir resultados del tensor
}

class TensorAudio(val context: Context) {  // Clase TensorAudio que recibe un contexto de Android

    private var modelFileName = "modelsound10.tflite"  // Nombre del archivo del modelo
    private val probabilityThreshold: Float = 0.4f  // Umbral de probabilidad

    lateinit var resultListener: TensorResultListener  // Variable para el listener de resultados

    fun setResultListener(listener: TensorResultListener) {  // Método para establecer el listener de resultados
        this.resultListener = listener  // Asigna el listener recibido
    }

    fun initialize() {  // Método para inicializar la funcionalidad del tensor
        // Carga el modelo seleccionado desde el archivo
        val audioClassifier = AudioClassifier.createFromFile(context, modelFileName)

        // Crea el tensor de entrada para audio
        val inputTensor = audioClassifier.createInputTensorAudio()

        // Crea el grabador de audio
        val audioRecord = audioClassifier.createAudioRecord()

        // Inicia la grabación
        audioRecord.startRecording()

        // Programa una tarea periódica
        Timer().scheduleAtFixedRate(1, 700) {
            // Carga el tensor con los datos de audio grabados
            inputTensor.load(audioRecord)

            // Realiza la clasificación
            val classifications = audioClassifier.classify(inputTensor)

            // Extrae los resultados relevantes
            val output = extractMaxResults(classifications)

            // Notifica al listener si hay resultados significativos
            if (output.isNotEmpty()) {
                resultListener.onTensorResult(output)
            }
        }
    }

    private fun extractMaxResults(classifications: MutableList<Classifications>): List<String> {
        // Filtra las categorías con puntuación superior al umbral de probabilidad
        val filteredResults = classifications[0].categories.filter {
            it.score > probabilityThreshold
        }
        // Ordena los resultados filtrados por puntuación descendente y los convierte en cadena
        return filteredResults.sortedByDescending { it.score }
            .joinToString(separator = "\n") { "${it.label},${it.score}" }
            .split(",")
    }
}
```


**4) Configuramos los permisos en el Manifest.**

```kotlin 
    <uses-permission android:name="android.permission.RECORD_AUDIO" />
```

**5) Configuramos la clase principal.**

**5.1) Solicitamos los permisos de usar el microfono.**

declaramos una variable que nos almacenara el estado del permiso del microfono.

```kotlin 
    //tiene permisos de microfono
    private var isMicPermiso = false
```

Llamamos a la funcion checkPermisoMic, que nos indicara si tiene permisos de usar el microfono 


```kotlin 
     //verifica permiso de microfono
        isMicPermiso = checkPermisoMic()
```
```kotlin 
       /**
     * Chekea si tiene permisos de microfono
     */
    private fun checkPermisoMic(): Boolean {
        val permiso = ContextCompat.checkSelfPermission(this,Manifest.permission.RECORD_AUDIO)
        return permiso == PackageManager.PERMISSION_GRANTED
    }
```

```kotlin 
    ' override fun onRequestPermissionsResult(
        requestCode: Int,
        permissions: Array<out String>,
        grantResults: IntArray
    ) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults)
        when(requestCode){
            REQUEST_RECORD_PERMISSION->{
                isMicPermiso = grantResults[0] == PackageManager.PERMISSION_GRANTED
                var mensaje = ""
                mensaje = if (isMicPermiso) "Permiso asignado" else "Permiso denegado"
                Toast.makeText(this, mensaje, Toast.LENGTH_SHORT).show()
            }
        }
    }
```

```kotlin 
      /**
     * Permisos de microfono
     */
    private fun permisosMic() {
        ActivityCompat.requestPermissions(
            this, arrayOf(Manifest.permission.RECORD_AUDIO),
            REQUEST_RECORD_PERMISSION
        )
    }
```

**5.2) Inicializamos la clase TensorAudio**

```kotlin 
     //init tensorAudio
        tensorAudio = TensorAudio(this)
```





**5.3) Configuramos la clase que contendra las acciones de las vistas. **
```kotlin 
     //Views
        viewStart()
```

```kotlin 
     /**
     * Configuraciones de vistas
     */
    private fun viewStart() {
        //TODO Boton escuchar
       vb.btnEscuchar.setOnClickListener {
           if(isMicPermiso){
                tensorAudio.initialize()
                //datos recibidos
                var datosRx = ""
                var mensajeCompleto = ""
                //no duplicar
                var isDuplicate = false
                //escuchar respuesta de tensorAudio
               tensorAudio.setResultListener(object :TensorResultListener{
                   override fun onTensorResult(results: List<String>) {
                       Log.d("outputModel",results[0])
                       when(results[0][0]){
                           '0'->{
                               if (!isDuplicate){
                                   datosRx+="0"
                                   isDuplicate = true
                               }
                           }
                           '1'->{
                               if (!isDuplicate){
                                   datosRx+='1'
                                   isDuplicate = true
                               }
                           }
                           '2'->isDuplicate = false
                       }

                       if (datosRx.length==8){
                           val charValue = datosRx.toInt(2).toChar()
                           mensajeCompleto+=charValue
                           //set mensaje consola
                           vb.txtConsolaRx.text = mensajeCompleto
                           //resetear datos Rx
                           datosRx = ""
                       }
                   }

               })

           }else{
               permisosMic()
           }
       }

        //TODO Boton enviar binarios
        vb.btnEnviarMensaje.setOnClickListener {
            if (vb.edtTextEnviar.text.isNotEmpty()){
                //texto a enviar
                val txtBinario = vb.edtTextEnviar.text.toString().toBinary()
                vb.txtEnviar.text = txtBinario
                //enviar texto con sonido
                thread(start = true){
                    for (binario in txtBinario){
                        when(binario){
                            '0'->playSound(R.raw.dos250)
                            '1'->playSound(R.raw.cuatro400)
                            else->{
                                Thread.sleep(tiempoPausa)
                            }
                        }
                    }
                }
            }
        }
    }
```

**5.4) Insertamos las frecuencias con las que fueron entrenados los modelos. **

**Frecuencia de 400hz:** https://cienciayculturacreativa.com/2024/comunicacion_ultrasonido/cuatro400.wav

**Frecuencia de 250hz:** https://cienciayculturacreativa.com/2024/comunicacion_ultrasonido/dos250.wav

en la carpeta raw

<a href="https://youtu.be/gwEq0-ACUr8?si=C6XIkI40u1sTYTNc&t=730">
  <img src="https://github.com/jose-jhr/Comunicacion_por_Ultrasonido/assets/66834393/246bdcaa-d87d-4f06-88d6-02f2860db04f" width="450">
</a>




```kotlin 
     //init tensorAudio
        tensorAudio = TensorAudio(this)
```







