# SwiftHtmlPdf

<p align="left">
<a href="https://cocoapods.org/pods/SwiftHtmlPdf"><img src="https://img.shields.io/cocoapods/v/SwiftHtmlPdf" alt="CocoaPods compatible" /></a>
<img src="https://img.shields.io/badge/platform-iOS-blue.svg?style=flat" alt="Platform iOS" />
<a href="https://developer.apple.com/swift"><img src="https://img.shields.io/badge/swift5-compatible-4BC51D.svg?style=flat" alt="Swift 5 compatible" /></a>
<a href="https://github.com/NGromann/SwiftHtmlPdf/blob/master/LICENSE"><img src="http://img.shields.io/badge/license-MIT-blue.svg?style=flat" alt="License: MIT" /></a>
</p>


Generate HTML and PDF documents by using template html files and filling them with your data.

This library allows you to generate HTML and PDF using HTML template files. Try out our [example project!](/Example)

## Requirements

* IOS 10 +

## Installation

You can install SwiftHtmlPdf using Cocoa Pods.

Add SwiftHtmlPdf into your project's `Podfile`:

```ruby
target 'MyApp' do
  pod 'SwiftHtmlPdf', '~> 1.0'
end
```

Then run the following command:

```bash
$ pod install
```

## Usage

### Create a html template resource and save it in your project
```html
<!DOCTYPE html>
<html>
	<head>
		<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
        <meta content="text/html; charset=utf-8" http-equiv="content-type">
		...
		</style>
	</head>
	<body>
        <h1>PDF Example</h1>
        <ul>
            <item name="MyListItem"/>
        </ul>
	</body>
</html>

<region name="MyListItem">
    <li>
        <field name="Name"/>
    </li>
</region>
```

Note the following HTML tags
* ```<item name="MyListItem"/>```
  * This is a reference to the *MyListItem* region.
* ```<region name="MyListItem">...</region>```
  * This is a blueprint for an instance of *MyListItem*
* ```<field name="Name"/>```
  * This is a field that will be replaced by a variable
  
### Fill the template with data
First create your model and implement ```PDFComposerDelegate```
```swift
import SwiftHtmlPdf
...
class MyListItem: PDFComposerDelegate {
    var name: String
    
    init(_ name: String) {
    	self.name = name
    }
    
    func valueForParameter(parameter: String, index: Int) -> String {
        switch parameter {
        case "Name":
            return name
        default:
            print("Unhandled PDF Key \(parameter) in MyListItem")
            return parameter
        }
    }
    
    func itemsForParameter(parameter: String, index: Int) -> [PDFComposerDelegate] {
        return []
    }
}
```

Now implement the delegate in your root object. In this case, the root object is the ViewController.
```swift
extension ViewController: PDFComposerDelegate {
    var myListItems = [MyListItem("Hello"), MyListItem("World")]
    
    func valueForParameter(parameter: String, index: Int) -> String {
    	return ""
    }
    
    func itemsForParameter(parameter: String, index: Int) -> [PDFComposerDelegate] {
        return myListItems
    }
```

Note the following delegate functions:
* ```valueForParameter(parameter: String, index: Int)```
	* This function is called for every ```<field name="{parameter}"/>``` in your template
* ```func itemsForParameter(parameter: String, index: Int) -> [PDFComposerDelegate]```
	* The function is called for every ```<item name="{parameter}"/>``` in the current region of the template
	* Return an array of child objects that will handle the region.
	* For every returned object, a new region will be instantiated: ```<region name="{parameter}">...</region>```

### Show a Preview Dialog in your app
```swift
func showPdfPreview() {
        let preview = PDFPreviewController.instantiate()
        
        do {
	    let resourceName = "planbuildpro-baukosten-template"
	    let delegate = self
            try preview.loadPreviewFromHtmlTemplateResource(templateResource: resourceName, delegate: delegate)

            present(preview, animated: true, completion: nil)
        } catch {
            print("Could not open pdf preview")
        }
    }
```
* resourceName is the file name of the [template html](#Create-a-html-template-resource-and-save-it-in-your-project). Do not include the suffix (.html)
* delegate is the View Controller (the root object)

### Parsing templates without the Preview Dialog
To parse the html template without the preview dialog, you can the following function:
```swift
let htmlContent = PDFComposer.renderHtmlFromResource(templateResource: templateResource, delegate: delegate)
```
This function works similar to the preview dialog but returns parsed html.

Next up you can transform the html content into a PDF:
```swift
let pdfData = PDFComposer.exportHTMLContentToPDF(HTMLContent: htmlContent)
```

Alternatively you can use the following function to create a pdf file:
```swift
let pdfData = PDFComposer.exportHTMLContentToPDFFile(HTMLContent: htmlContent, path: path)
```

Now you can share the pdf using a ```UIActivityViewController```:
```swift
let activityVC = UIActivityViewController(activityItems: [pdfData], applicationActivities: nil)
        
self.present(activityVC, animated: true, completion: nil)
```
