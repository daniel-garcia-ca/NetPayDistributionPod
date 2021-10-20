<br/>
<p style="background:blue; padding:20px; display:flex; justify-content: center;  margin-top:10px">

<img heigth="200px" width="550px" center src="https://github.com/netpaymx/NetPaySDKPod/blob/master/img/netpay-logo-white.png?raw=true"/>
<br>

</p>

<br>
<h3>
Resumen
</h3>

<ol>
    <li>Requerimientos</li>
    <li>Integración de Pod</li>
    <li>Construcción de formulario de cobro.</li>
    <li>Personalización de formulario de cobro.</li>
    <li>Componentes para implementar en formualrio personalizado.</li>
    <li>Implementación de metodo para generar token.</li>
</ol>

**Integración de el SDK de NetPay IOS a tu aplicación de Custom Checkout por medio del gestor de dependencias Cocoapods.**

<br>

<img src="https://img.shields.io/static/v1?label=Swift&message=5.0,5.1&color=orange"/>   <img src="https://img.shields.io/static/v1?label=Plataforms&message=IOS&color=yellowgreen"/>  <img src="https://img.shields.io/static/v1?label=Pod&message=v1.0.1&color=blue"/>  <img src="https://img.shields.io/static/v1?label=Swift Package Manager&message=Compatible&color=orange"/> <img src="https://img.shields.io/static/v1?label=IOS Minimo &message=8.0&color=critical"/>

<h2>1.Requerimientos</h2>
<ul>
    <li>Netpay API public key.</li>
    <li>iOS 8 o un target de implementación superior.</li>
    <li>Xcode 12.2 o superior.</li>
    <li>Swift 5.0 o superior (Swift 5.1)</li>
</ul>
<h2>2.Integración de POD</h2>

**CocoaPods:**

```swift
source 'https://github.com/CocoaPods/Specs.git'
    
    use_frameworks!
    
    target 'target' do
        use_frameworks!
        pod 'NetPaySDK', '~> 1.0.1'
    end
```

<h2>3. Construcción de formulario de cobro.</h2>
<p>NetPay iOS SDK proporciona formularios de interfaz de usuario fáciles de usar tanto para tokenizar una tarjeta de crédito como para crear una fuente de pago que se pueda integrar fácilmente en su aplicación.</p>
<h3>3.1 Uso de formulario de tarjeta.</h3>
<p>Para usar el controlador en su aplicación, modifique su controlador de vista con las siguientes adiciones:</p>
<ul>
	<li>Importar libreria netpay</li>
</ul>

```swift
 import UIKit
 import NetPaySDK
```

<ul>
    <li>Asignar llave pública (public key)</li>
</ul>

```swift
 private let publicKey = "pk_netpay_kSjXddOJMPuxfqEsEICyIOKUs"
 private let testMode = true
```
<ul>
    <li>Asignar el identificador</li>
</ul>

```swift
override func shouldPerformSegue(withIdentifier identifier: String, sender: Any?) -> Bool {
            if identifier == "PresentCreditFormWithModal" ||
                identifier == "ShowCreditForm" {
                return currentCodePathMode == .storyboard
            }
	
            return true
        }
```

<ul>
    <li>Asignar la llave pública y testMode al controlador del formulario de pago</li>
</ul>

```swift
var halfModalTransitioningDelegate: HalfModalTransitioningDelegate?

override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
            super.prepare(for: segue, sender: sender)
    
            if segue.identifier == "PresentCreditFormWithModal",
                let creditCardFormNavigationController = segue.destination as? UINavigationController,
                let creditCardFormController = creditCardFormNavigationController.topViewController as? CreditCardFormViewController {
                creditCardFormController.publicKey = publicKey
                creditCardFormController.testMode = testMode
                creditCardFormController.handleErrors = true
                creditCardFormController.delegate = self
				
		self.halfModalTransitioningDelegate = HalfModalTransitioningDelegate(viewController: self, presentingViewController: segue.destination)
            
            	segue.destination.modalPresentationStyle = .custom
            	segue.destination.transitioningDelegate = self.halfModalTransitioningDelegate
				
            } else if segue.identifier == "ShowCreditForm",
                let creditCardFormController = segue.destination as? CreditCardFormViewController {
                creditCardFormController.publicKey = publicKey
                creditCardFormController.testMode = testMode
                creditCardFormController.handleErrors = true
                creditCardFormController.delegate = self
            }
        }
```

<ul>
    <li>Crear modal de formulario</li>
</ul>

```swift
        @IBAction func showModalCreditCardForm(_ sender: Any) {
            guard currentCodePathMode == .code else {
                return
            }
            let creditCardFormController = CreditCardFormViewController.makeCreditCardFormViewController(withPublicKey: publicKey)
            creditCardFormController.handleErrors = true
            creditCardFormController.delegate = self
            let navigationController = UINavigationController(rootViewController: creditCardFormController)
            present(navigationController, animated: true, completion: nil)
        }
```

<ul>
    <li>Crear formulario push</li>
</ul>

```swift
        @IBAction func showCreditCardForm(_ sender: UIButton) {
            guard currentCodePathMode == .code else {
                return
            }
            let creditCardFormController = CreditCardFormViewController.makeCreditCardFormViewController(withPublicKey: publicKey)
            creditCardFormController.handleErrors = true
            creditCardFormController.delegate = self
            show(creditCardFormController, sender: self)
        }
```

<ul>
    <li>Delegate para View Controller del formulario de tarjeta</li>
</ul>

```swift
extension ViewController: CreditCardFormViewControllerDelegate {
        func creditCardFormViewControllerDidCancel(_ controller: CreditCardFormViewController) {
            dismissForm()
        }
        //Success
        func creditCardFormViewController(_ controller: CreditCardFormViewController, didSucceedWithToken token: Token) {
            dismissForm(completion: {
                let alertController = UIAlertController(
                    title: "Token Creado",
                    message: "El token: \(token.token) fué creado satisfactoriamente. Por favor envía el token a tu back-end para realizar el checkout.",
                    preferredStyle: .alert
                )
                let okAction = UIAlertAction(title: "OK", style: .cancel, handler: nil)
                alertController.addAction(okAction)
                self.present(alertController, animated: true, completion: nil)
            })
        }
        //Error
        func creditCardFormViewController(_ controller: CreditCardFormViewController, didFailWithError error: Error) {
            dismissForm(completion: {
                let alertController = UIAlertController(
                    title: "Error",
                    message: error.localizedDescription,
                    preferredStyle: .alert
                )
                let okAction = UIAlertAction(title: "OK", style: .cancel, handler: nil)
                alertController.addAction(okAction)
                self.present(alertController, animated: true, completion: nil)
            })
        }
    }
```

<li>Personalización de formulario de cobro.</li>
Se crea una variable con  la instancia del objeto StyleFormBuilder y se agrega en la creacion del formualrio ya sea en modal o push.

```swift
let styleForm = StyleFormBuilder()
                    .addBackgroundColor(color: UIColor.white)
                    .addBackgroundColorButton(color: UIColor.blue)
                    .addTextButton(text: "Texto boton pagar")
                    .addTintColorButtonCVVInfo(color: UIColor.lightGray)
                    .addFieldColor(color: UIColor.black)
                    .addPlaceholderTextColor(color: UIColor.lightGray)
                    .addBorderColorBoxField(color: UIColor.black)
                    .build()
                
creditCardFormController.styleForm = styleForm
```

<li>Propiedades de builder para configurar diseño de formulario.</li></br>

| Metodo | Descripción |
| --- | --- |
| addBackgroundColor(color: UIColor) | Permite cambiar el color de fondo del formulario. |
| addBackgroundColorButton(color: UIColor) | Permite cambiar el color de fondo del boton. |
| addTintColorButtonCVVInfo(color: UIColor) | Cambia el color del icono de cvv. |
| addFieldColor(color: UIColor) | Cambia el color de los labels del formulario. |
| addPlaceholderTextColor(color: UIColor) | Cambia el color de texto de los placeholder de los UITextField. |
| addSizeFieldTitleFont(size: CGFloat) | Cambia el tamaño del titulo del formulario. |
| addSizeFieldsFont(size: CGFloat) | Cambia el tamaño de todos los UITextField del formulario. |
| addSizeBorderBoxFields(size: CGFloat) | Cambia el tamaño del borde de todos los UITextField del formulario. |
| addCustomFieldTitleFont(font: UIFont) | Cambia el estilo del texto de el titulo del formulario. |
| addCustomFieldFont(font: UIFont) | Cambia el estilo del texto de todos los UITextField del formulario. |
| addCustomPlaceholderFont(font: UIFont) | Cambia el estilo del texto de todos los UITextField que contienen placeholder del formulario. |
| addTextButton(text: String) | Cambia el texto del boton del formulario. |
| addTxtLabelNumCard(text: String) | Cambia el texto del label de número de tarjeta del formulario. |
| addTxtLabelDateExpiration(text: String) | Cambia el texto del label de fecha de expiración del formulario. |
| addTxtLabelCvv(text: String) | Cambia el texto del label de número de CVV del formulario. |
| addCornerLabelBox(corner: CGFloat) | Cambia el tamaño redondeado de las esquinas de todos los componentes UITextField del formulario. |
| addCornerButton(corner: CGFloat) | Cambia el tamaño redondeado de las esquinas del boton del formulario. |
| addBorderColorButton(color: UIColor) | Agrega color al borde del boton del formulario. |
| addBorderWithButton(border: CGFloat) | Agrega el tamaño del borde en el boton del formulario. |
| addBackgroundBoxField(color: UIColor) | Cambia el color de fondo de todos los componentes UITextField en el formulario. |
| addShadowColorButton(color: UIColor) | Agrega el color de la sombra en el boton del formulario. |
| addShadowOffsetButton(offset: CGSize) | Agrega el desplazamiento (en puntos) de la sombra de la capa en el boton del formulario. |
| addShadowRadiusButton(radius: CGFloat) | Agrega el radio de desenfoque (en puntos) utilizado para representar la sombra de la capa en el boton del formulario. |
| addShadowOpacityButton(opacity: Float) | Agraga la opacidad de la sombra en el boton dl formulario. |
| addMasksToBoundsButton(maskToBounds: Bool) | Booleano que indica si las subcapas están recortadas a los límites de la capa en los componentes UITextField del formulario. |
| addShadowColorBoxFields(color: UIColor) | Agrega el color de la sombra en los componentes UITextField del formulario. |
| addShadowOpacityBoxFields(opacity: Float) | Agraga la opacidad de la sombra en los componentes UITextField del formulario. |
| addShadowRadiusBoxFields(radius: CGFloat) | Agrega el radio de desenfoque (en puntos) utilizado para representar la sombra de la capa en los componentes UITextField del formulario. |
| addShadowOffsetBoxFields(offset: CGSize) | Agrega el desplazamiento (en puntos) de la sombra de la capa en los componentes UITextField del formulario. |
| addMasksToBoundsFileds(maskToBounds: Bool) | Booleano que indica si las subcapas están recortadas a los límites de la capa en los componentes UITextField del formulario. |
| addTitle(title: String) | Cambia el titulo del formulario. |
| addTxtButtonColor(color: UIColor) | Cambia el color del texto en el boton del formulario. |

<li>Componentes para implementar en formualrio personalizado.</li>
<br/>
El SDK provee tres componentes para ingresar datos de la tarjeta:<br/>

<ul>
    <li>NumberTextFieldExpose</li>
    <li>ExpiryDateTextFieldExpose</li>
    <li>CVVTextFieldExpose</li>
</ul>

<p>Estos componentes exitienden del objeto UITextField se pueden agregar en el fomulario de manera programatica o desde el storyboar.</p>
<p>Nota: Al obtener el valor de la propiedad text este valor se regresa encriptado por seguridad y proteccion de información, solo el sdk puede encriptar y desencriptar el valor.</p>

<li>Implementación de metodo para generar token.</li><br/>

```swift
let netPaySDKInit = NetPaySDKInit(publicKey: self.publicKey, testMode: self.testMode, protocoloResponseToken: self)

netPaySDKInit.requestTokenBuild(numberCardEncrypted: String, expMothnEncrypted: String, expYearEncrypted: String, cvvEncrypted: String)
```

Los valores numberCardEncrypted, expMothnEncrypted, expYearEncrypted, cvvEncrypted argumentos en el metodo requestTokenBuild solo se puden obtener mediante los componentes antes descritos.
