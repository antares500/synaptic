synaptic [![Build Status](https://travis-ci.org/cazala/synaptic.svg?branch=master)](https://travis-ci.org/cazala/synaptic)
========

Synaptic es una biblioteca de redes neuronales Javascript para **Node.js** y el **navegador**, su algoritmo generalizado está libre de la arquitectura, para que pueda construir y formar básicamente cualquier tipo de primer orden o incluso con arquitecturas de [redes neuronales de segundo orden](http://en.wikipedia.org/wiki/Recurrent_neural_network#Second_Order_Recurrent_Neural_Network).

Esta biblioteca incluye algunas arquitecturas integradas como [perceptrones multicapa](http://en.wikipedia.org/wiki/Multilayer_perceptron), [memoria de múltiples capas a largo corto plazo](http://en.wikipedia.org/wiki / Long_short_term_memory) redes (LSTM), [máquinas de estado líquido](http://en.wikipedia.org/wiki/Liquid_state_machine) o [Hopfield](http://en.wikipedia.org/wiki/Hopfield_network) redes, y un entrenador capaz de formar una red dada, que incluye una función de tareas de formación/pruebas como la solución de un XOR, completar una tarea Secuencia distraída Recall o un [Embedded Reber Gramática](http://www.willamette.edu/~gorr/clases/cs449/reber.html) de prueba, por lo que puede probar y comparar el rendimiento de diferentes arquitecturas fácilmente.


El algoritmo implementado por esta biblioteca se ha tomado de papel de Derek D. Monner:

[Un algoritmo de entrenamiento LSTM similar generalizada de redes neuronales recurrentes de segundo orden](http://www.overcomplete.net/papers/nn2012.pdf)

Hay referencias a las ecuaciones en que el papel comentado a través del código fuente.

####Introducción

Si no tienes conocimiento previo acerca de redes neuronales, debe comenzar por [leer esta guía](https://github.com/cazala/synaptic/wiki/Neural-Networks-101).


Si quieres un ejemplo práctico sobre cómo alimentar datos a una red neuronal, a continuación, echar un vistazo a [este artículo](https://github.com/cazala/synaptic/wiki/Normalization-101).

También es posible que desee echar un vistazo a [este artículo] (http://blog.webkid.io/neural-networks-in-javascript/).

####Demostraciones

- [Resolver un XOR](http://synaptic.juancazala.com/#/xor)
- [Discreta secuencia de tareas Recall](http://synaptic.juancazala.com/#/dsr)
- [Aprender filtros de imagen](http://synaptic.juancazala.com/#/image-filters)
- [Pintar una imagen](http://synaptic.juancazala.com/#/paint-an-image)
- [Mapa autoorganizado](http://synaptic.juancazala.com/#/self-organizing-map)
- [Leer de Wikipedia](http://synaptic.juancazala.com/#/wikipedia)

El código fuente de estas demostraciones se pueden encontrar en [esta rama](https://github.com/cazala/synaptic/tree/gh-pages/scripts).

####Empezando

- [Neuronas](https://github.com/cazala/synaptic/wiki/Neurons/)
- [Capas](https://github.com/cazala/synaptic/wiki/Layers/)
- [Redes](https://github.com/cazala/synaptic/wiki/Networks/)
- [Entrenador](https://github.com/cazala/synaptic/wiki/Trainer/)
- [Arquitectura](https://github.com/cazala/synaptic/wiki/Architect/)


##Visión de conjunto

###Instalación

#####Node.js

Se puede instalar con sináptica [MNP](http://npmjs.org):

```cmd
npm install synaptic --save
```

#####En el navegador

Se puede instalar con sináptica [glorieta](http://bower.io):

```cmd
bower install synaptic
```

O simplemente puede usar el enlace de CDN, proporcionado amablemente por [CDNjs] (https://cdnjs.com/)

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/synaptic/1.0.4/synaptic.min.js"></script>
```

###Uso

```javascript
var synaptic = require('synaptic'); // this line is not needed in the browser
var Neuron = synaptic.Neuron,
	Layer = synaptic.Layer,
	Network = synaptic.Network,
	Trainer = synaptic.Trainer,
	Architect = synaptic.Architect;
```

Ahora puede empezar a crear redes, entrenarlos, o utilizar las redes construidas a partir de la [Arquitecto](http://github.com/cazala/synaptic#architect).

### Tareas Gulp

- **gulp**: runs all the tests and builds the minified and unminified bundles into `/dist`.
- **gulp build**: builds the bundle: `/dist/synaptic.js`.
- **gulp min**: builds the minified bundle: `/dist/synaptic.min.js`.
- **gulp debug**: builds the bundle `/dist/synaptic.js` with sourcemaps.
- **gulp dev**: same as `gulp debug`, but watches the source files and rebuilds when any change is detected.
- **gulp test**: runs all the tests.

###Ejemplos

#####Perceptron

Se trata de cómo se puede crear un simple **perceptrón**:

![perceptron](http://www.codeproject.com/KB/dotnet/predictor/network.jpg).

```javascript
function Perceptron(input, hidden, output)
{
	// create the layers
	var inputLayer = new Layer(input);
	var hiddenLayer = new Layer(hidden);
	var outputLayer = new Layer(output);

	// connect the layers
	inputLayer.project(hiddenLayer);
	hiddenLayer.project(outputLayer);

	// set the layers
	this.set({
		input: inputLayer,
		hidden: [hiddenLayer],
		output: outputLayer
	});
}

// extend the prototype chain
Perceptron.prototype = new Network();
Perceptron.prototype.constructor = Perceptron;
```

Ahora puede probar su nueva red mediante la creación de un entrenador y enseñar el perceptrón para aprender un XOR

```javascript
var myPerceptron = new Perceptron(2,3,1);
var myTrainer = new Trainer(myPerceptron);

myTrainer.XOR(); // { error: 0.004998819355993572, iterations: 21871, time: 356 }

myPerceptron.activate([0,0]); // 0.0268581547421616
myPerceptron.activate([1,0]); // 0.9829673642853368
myPerceptron.activate([0,1]); // 0.9831714267395621
myPerceptron.activate([1,1]); // 0.02128894618097928
```

#####Memoria a Largo Corto Plazo

Se trata de cómo se puede crear un simple memoria a largo ** ** corto plazo a la red con la puerta de entrada, olvidar puerta, puerta de salida, y las conexiones de mirilla:

![Memoria a largo corto plazo] (http://people.idsia.ch/~juergen/lstmcell4.jpg)

```javascript
function LSTM(input, blocks, output)
{
	// create the layers
	var inputLayer = new Layer(input);
	var inputGate = new Layer(blocks);
	var forgetGate = new Layer(blocks);
	var memoryCell = new Layer(blocks);
	var outputGate = new Layer(blocks);
	var outputLayer = new Layer(output);

	// connections from input layer
	var input = inputLayer.project(memoryCell);
	inputLayer.project(inputGate);
	inputLayer.project(forgetGate);
	inputLayer.project(outputGate);

	// connections from memory cell
	var output = memoryCell.project(outputLayer);

	// self-connection
	var self = memoryCell.project(memoryCell);

	// peepholes
	memoryCell.project(inputGate);
	memoryCell.project(forgetGate);
	memoryCell.project(outputGate);

	// gates
	inputGate.gate(input, Layer.gateType.INPUT);
	forgetGate.gate(self, Layer.gateType.ONE_TO_ONE);
	outputGate.gate(output, Layer.gateType.OUTPUT);

	// input to output direct connection
	inputLayer.project(outputLayer);

	// set the layers of the neural network
	this.set({
		input: inputLayer,
		hidden: [inputGate, forgetGate, memoryCell, outputGate],
		output: outputLayer
	});
}

// extend the prototype chain
LSTM.prototype = new Network();
LSTM.prototype.constructor = LSTM;
```

Estos son ejemplos de las explicaciones, la [Arquitectura](https://github.com/cazala/synaptic/wiki/Architect/) ya incluye múltiples capas de perceptrones y arquitecturas de red multicapa LSTM.

##Contribuir

**Synaptic** es un proyecto Open Source que se inició en Buenos Aires, Argentina. Cualquiera en el mundo es bienvenido a contribuir al desarrollo del proyecto.

Si quieres contribuir dude en enviar respuestas, sólo asegúrese de que todo se ejecuta por defecto **gulp** antes de enviar. De esta manera se encontrará con todas las especificaciones de prueba y construir los archivos de distribución web.

<3
