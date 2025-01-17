# README.md

Integrantes:
 Galiano, Sergio Andrés | 405369
 Gattas, Agustín | 405298
 Konecny Calo, Julián Gabriel | 405795
 Oliva, Maximiliano Agustín | 405193
 Rossi, Braian Alexis | 114080
 Villegas, Juan Cruz | 405601

SERGIO: Aspectos a tener en cuenta al agregar programación:

FACTORY: 
	* Cada clase nueva creada por nosotros, debe instanciarse por factory. La clase abstracta AbstractFactory en Servicios de Acceso a Datos
	  no hace falta modificarla. Tiene un método abstracto común que sirve para cualquier objeto.
	  Lo que debemos hacer, es crearle una entrada en ModeloFactory si estamos trabajando con modelos (dominios), o en ServiciosFactory si trabajamos
	  en el cliente local(para crear por ejemplo una referencia a una clase gestora).
	  Ej.: Articulo nuevo_articulo = (Articulo)factoria_modelos.CreaObjeto("articulo"); El tipo debe castearse, dado que la factoría devuelve un tipo object para 
	  que pueda ser genérico (de lo contrario deberíamos hacer un abstract para cada clase).
	  Entre paréntesis va el nombre que queramos para referirnos a esa clase (puede ser el mismo nombre en minúsculas). En en ServiceFactory o ModeloFactory, agregar una entrada
	  al switch para capturar la lammada, y retornar el constructor con los parámetros que deseemos. Podemos llamar mas de una vez a la creación del mismo objeto
	  si necesitamos pasarle distinta cantidad de parámetros.. Ej. al utilizar el FormBuscador, lo llamamos pasando por ej. buscador_articulos y en 
	  el switch lo creamos con los parámetros que necesitemos. Otro podrá usar buscador_clientes y lo creará con otros parámetros.
	  TRABAJAMOS CON DOS FACTORY UNO QUE ES INSTANCIA DE MODELOFACTORY PARA LOS MODELOS Y OTRO QUE ES INSTANCIA DE SERVICIOSFACTORY QUE ES PARA LOS FORMULARIOS 
	  Y OTRAS CLASES A NIVEL DE PRESENTACION.


DATOS:
	* El DBHelper está con patrón singleton, con lo cual tiene un método ObtenerInstancia para instanciarlo u obtener la ya creada.
	* Dispnone de los siguientes métodos genéricos: (cubrirían todo el espectro de lo que hay que hacer) 
		- AbreConexionConTransacción: abre una conexión y una transacción;
		- CierraConexionConTransacción: cierra una conexión y hace el commit de la transacción.
		- CargarTabla: para cargar datos dentro de un DataTable y cargar un combo, por ejemplo. Trabaja solamente con SP.
		- EjecutaComando: ejecuta cualquier otro comando SQL y dispone de una lista de parámetros de entrada y un parámetro de salida en 
		  caso de ser necesario. También trabaja solamente con SP.
		- El uso para EjecutaComando debería ser de la siguiente forma:
			DBHelper.ObtenerInstancia().AbreConexionConTransaccion();
			Crear una lista de SQLParameters para pasarle al método EjecutaComando o nada si utilizamos CargarTabla
			ejecutar el comando por Ej. int resultado = DBHelper.ObtenerInstancia().EjecutaComando("SP_ARTICULOS_ALTA", param_articulo, null);
			Si es maestro detalle ejecutar otro u otros comandos.
			DBHelper.ObtenerInstancia().CierraConexionConTransaccion();
		- El uso de CargarTabla debería ser de la siguiente forma:
			SOLAMENTE
			nueva_tabla = DBHelper.ObtenerInstancia().CargarTabla("SP_ARTICULOS_CARGA_TIPOS"); 
			No hace falta abrir ni cerrar transacción aquí. El método abre la conexion y la cierra luego de leer la tabla.
	* Nuestro objeto DAO, debería derivar de la interfaz IObjetoDAO que tiene algunos métodos comunes para CRUD, pero le podemos agregar propios
	  que no estén en la interface (ver DAOArticulo)

API:
	* Debemos crear los controllers necesarios para cada tema, por ejemplo, uno para proveedores, otro para clientes, otro para empleados, etc.
	* Es importante declarar bien la ruta en el decorador. Por ejempo si nuestro controller se llama Articulos y en el decorador del controller
	  colocamos [Route("api/[controller]")], la ruta principal a llamar será "/api/Articulos". Si luego creamos un método Obtner_Lista_Simple con 
	  un decorador [HttpGet, Route("Obtener_Lista_Simple")], la ruta total a llamar desde el cliente será "/api/Articulos/Obtener_Lista_Simple"

CLIENTE (FARMACEUTICA):
	* Se crea con patrón singleton para HTTPCLIENT. Se llama igual que el DBHelper. Por ej: 
	  string contenido = await ClientSingleton.GetInstance().GetAsync("/api/Articulos/Obtener_Tipo_Articulos");
	* Todos los métodos en el gestor que llaman a la API deben ser declarados como async y de tipo Task<alguna_clase> y para llamar a la api se utiliza await
	  Ej:
	    public async Task<List<Tipo_Articulo>> GetTipoArticulos()
        {
            List<Tipo_Articulo> lista_tipos = new List<Tipo_Articulo>();
            string contenido = await ClientSingleton.GetInstance().GetAsync("/api/Articulos/Obtener_Tipo_Articulos");
            if (contenido != string.Empty)
                lista_tipos = JsonConvert.DeserializeObject<List<Tipo_Articulo>>(contenido); ;
            return lista_tipos;
        }
		Los comandos Deserialize y Serialize convierten automáticamente el json en el objeto indicado (obviamente, deben matchear todos los campos)

CONTROL NUMBERTEXTBOX
	* Lo pueden usar como si fuera un textbox (de hecho hereda del textbox). Lo arrastran del cuadro de herramientas al formulario.
	* No acepta ningun caracter que no sean números, el signo menos (si AceptaNegativos = true) y el punto decimal en caso de ser con coma.
	* Propiedades nuevas que agrega:
		- AceptaNegativos (true or false). Si se va a poder ingresar o no un múmero negativo
		- AceptaValorCero (true or false). Si se va a poder ingresar un cero como valor.
		- CerosIzquierda: (true or false). Para el caso de enteros, muestra el resultado como: 000005252 por ejemplo.
		- Decimales: la cantidad de decimales con los que trabaja.
		- DecimalesConFormato: cantidad de decimales que muestra al salir del campo. Si decimales = 4 y DecimalesConFormato = 2, cuando se entre al campo va a mostrar
		  652,2552 y cuando salga del campo va a mostrar 652,26 (muestra dos pero trabaja con 4). Normalmente lo mas común es que estos parámetros sean ambos iguales.
		- Digitos: cantidad de digitos a la izquierda de la coma.
		- FormatoNegativo (signo o paréntesis). Si va a mostrar -652 ó (652).
		- SimboloMoneda: utiliza el símbolo que coloquemos allí.
		- TipoNumerico:
			* Entero: sin decimales. Decimales y DecimalesConFormato deberían ser CERO.
			* Moneda: importes monetarios. Se muestra el símbolo de moneda antes del importe. Se pueden definir Decimales y DecimalesConFormato. 
				Se muestra con coma y separador de miles. Si el símbolo se muestra atrás, cambiar el formato en WINDOWS (opciones regionales de idioma).
			* Fijo: Programación no verificada. NO USAR.
			* Porcentaje: muestra un símbolo de porcentaje detrás. Se pueden definir Decimales y DecimalesConFormato. 
				Si el porcentaje mostrado es 5,25%, la propiedad ValorDecimal adquiere el valor de 0.0525
			* Estandar: es un número común, con decimales, sin símbolo de moneda: por ej. 1256625.38. Se muestra con coma y separador de miles.
			* CUIT: formatea como un número de CUIT con los guiones y además verifica que sea una CUIT válida, según el algoritmo de AFIP. Es un entero.
			* ValorEntero: ES LA PROPIEDAD QUE CAPTURA EL VALOR DEL CAMPO cuando seleccionamos entero o CUIT. Es de tipo long (es cero cuando el tipo es decimal)
			* ValorDecimal: ES LA PROPIEDAD QUE CAPTURA EL VALOR DEL CAMPO cuando seleccionamos algún otro formato (Moneda, Porcentaje, Estandar).
				Es de tipo double. (es cero cuando el tipo es entero)
			IMPORTANTE: No usar la propiedad TEXT del control para validar ingreso de caracteres.
						Solamente se usa para verificar si se cargó algo por ej. if(ntbxxx.text == string.empty) --> validar.
						No hace falta validar otra cosa, dado que el control NO recibe literales. Tampoco deja hacer un paste de literales dentro. 
						Sí se pueden copiar y pegar importes o valores de NumberTextBox con el mismo formato (entero a entero) (decimal a decimal).
			PARA ASIGNAR UN VALOR POR CÓDIGO USAR ntbCodigo_XXX.Text = varcodigo.ToString();

