# Flutter Quill Delta from HTML

This is a Dart package that converts HTML input into Quill Delta format, which is used in the flutter_quill package. This allows developers to easily convert HTML content to a format that can be displayed and edited using the Quill rich text editor in Flutter applications.

**This package** supports the conversion of a wide range of HTML tags and attributes into their corresponding Delta operations, ensuring that your HTML content is accurately represented in the Quill editor.

## Supported tags

```html
    Text Formatting
        <b>, <strong>: Bold text
        <i>, <em>: Italic text
        <u>, <ins>: Underlined text
        <s>, <del>: Strikethrough text
        <sup>: Superscript text
        <sub>: Subscript text

    Headings
        <h1> to <h6>: Headings of various levels

    Lists
        <ul>: Unordered lists
        <ol>: Ordered lists
        <li>: List items

    Links
        <a>: Hyperlinks with support for the href attribute

    Images
        <img>: Images with support for the src, alt, and width attributes

    Videos 
        <iframe>, <video>: Videos with support for the src

    Blockquotes
        <blockquote>: Block quotations

    Code Blocks
        <pre>, <code>: Code blocks

    Text Alignment
        <p style="text-align:left|center|right|justify">: Paragraph alignment

    Text attributes
        <p style="line-height: 1.0;font-size: 12;font-family: Times New Roman;color:#ffffff">: Inline attributes

    Custom Blocks (alternative to this package to create html to `CustomBlockEmbed` for Quill Js) 
```

## Not supported tags

```html
  Text indent
  <p style="padding: 10px">: indented paragraph
```

Getting Started

Add it to your pubspec.yaml:

```yaml
dependencies:
  flutter_quill_delta_from_html: ^1.1.6
```

Then, import the package and use it in your Flutter application:

```dart
import 'package:flutter_quill_delta_from_html/flutter_quill_delta_from_html.dart';

void main() {
  String htmlContent = "<p>Hello, <b>world</b>!</p>";
  var delta = HtmlToDelta().convert(htmlContent);
/*
   { "insert": "hello, " },
   { "insert": "world", "attributes": {"bold": true} },
   { "insert": "!" },
   { "insert": "\n" }
*/
}
```

## Creating your own `CustomHtmlPart` (alternative to create `CustomBlockEmbeds` from custom html)

First you need to define your own `CustomHtmlPart`

```dart
import 'package:flutter_quill_delta_from_html/flutter_quill_delta_from_html.dart';
import 'package:html/dom.dart' as dom;

/// Custom block handler for <pullquote> elements.
class PullquoteBlock extends CustomHtmlPart {
  @override
  bool matches(dom.Element element) {
    return element.localName == 'pullquote';
  }

  @override
  List<Operation> convert(dom.Element element, {Map<String, dynamic>? currentAttributes}) {
    final Delta delta = Delta();
    final Map<String, dynamic> attributes = currentAttributes != null ? Map.from(currentAttributes) : {};

    // Extract custom attributes from the <pullquote> element
    final author = element.attributes['data-author'];
    final style = element.attributes['data-style'];

    // Apply custom attributes to the Delta operations
    if (author != null) {
      delta.insert('Pullquote: "${element.text}" by $author', attributes);
    } else {
      delta.insert('Pullquote: "${element.text}"', attributes);
    }

    if (style != null && style.toLowerCase() == 'italic') {
      attributes['italic'] = true;
    }

    delta.insert('\n', attributes);

    return delta.toList();
  }
}
```

After, put your `PullquoteBlock` to `HtmlToDelta` using the param `customBlocks`

```dart
import 'package:flutter_quill_delta_from_html/flutter_quill_delta_from_html.dart';

void main() {
  // Example HTML snippet
  final htmlText = '''
    <html>
      <body>
        <p>Regular paragraph before the custom block</p>
        <pullquote data-author="John Doe" data-style="italic">This is a custom pullquote</pullquote>
        <p>Regular paragraph after the custom block</p>
      </body>
    </html>
  ''';

  // Registering the custom block
  final customBlocks = [PullquoteBlock()];

  // Initialize HtmlToDelta with the HTML text and custom blocks
  final converter = HtmlToDelta(customBlocks: customBlocks);

  // Convert HTML to Delta operations
  final delta = converter.convert(htmlText);
/*
This should be resulting delta
  {"insert": "Regular paragraph before the custom block\n"},
  {"insert": "Pullquote: \"This is a custom pullquote\" by John Doe", "attributes": {"italic": true}},
  {"insert": "\n"},
  {"insert": "Regular paragraph after the custom block\n"}
*/
}
```

## HtmlOperations

The `HtmlOperations` class is designed to streamline the conversion process from `HTML` to `Delta` operations, accommodating a wide range of `HTML` structures and attributes commonly used in web content.

To utilize `HtmlOperations`, extend this class and implement the methods necessary to handle specific `HTML` elements. Each method corresponds to a different `HTML` tag or element type and converts it into Delta operations suitable for use with `QuillJS`.

```dart
abstract class HtmlOperations {
  const HtmlOperations();
  //You don't need to override this method 
  //as it simply calls the other methods 
  //to detect the type of HTML tag
  List<Operation> resolveCurrentElement(dom.Element element);

  List<Operation> brToOp(dom.Element element);
  List<Operation> headerToOp(dom.Element element);
  List<Operation> listToOp(dom.Element element);
  List<Operation> paragraphToOp(dom.Element element);
  List<Operation> linkToOp(dom.Element element);
  List<Operation> spanToOp(dom.Element element);
  List<Operation> imgToOp(dom.Element element);
  List<Operation> videoToOp(dom.Element element);
  List<Operation> codeblockToOp(dom.Element element);
  List<Operation> blockquoteToOp(dom.Element element);
  bool isInline(String tagName);
  void processNode(dom.Node node, Map<String, dynamic> attributes, Delta delta);
  List<Operation> inlineToOp(dom.Element element);
}
```

## Contributions

If you find a bug or want to add a new feature, please open an issue or submit a pull request on the GitHub repository.

This project is licensed under the MIT License - see the [LICENSE](https://github.com/CatHood0/flutter_quill_delta_from_html/blob/Main/LICENSE) file for details.
