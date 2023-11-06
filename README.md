
We are going to create a project that parses a very simple markdown format while the user types into a text area and displays the resulting web page alongside it.

we are going to concentrate on formatting the first three header types, the horizontal rule, and paragraphs.

here is the project overview

 * We are going to create an application to parse markdown* The user will type into a text area
 * Every time the text area changes, we will parse the entire document again
 * We will break the document down based on where the user presses the Enter key
 * The opening characters will determine whether or not the line is markdown
 * Entering # followed by a space is replaced by an H1 heading
 * Entering ## followed by a space is replaced by an H2 heading
 * Entering ### followed by a space is replaced by an H3 heading
 * Entering --- is replaced by a horizontal rule
 * If the line does not start with markdown, the line is treated as a paragraph
 * The resulting HTML will be displayed in a label
 * If the content in the markdown text area is empty, the label will contain an empty paragraph
 * The layout will be done in Bootstrap and the content will stretch to 100% height

*** Our starting point is this page, which stretches across the full width of the screen by setting the container to use container-fluid, 
and divides the interface into two equal parts by setting col-lg-6 on both sides:

<div class="container-fluid">
  <div class="row">
    <div class="col-lg-6">
    </div>
    <div class="col-lg-6">
    </div>
  </div>
</div>

*** When we add our text area and label components to our form, we find that rendering them in this row does not automatically expand them to fill the height of the screen. 
We need to make a couple of adjustments. 
First, we need to manually set the style of the html and body tags to fill the available space. To do this, we add the following in the header:

<style>
  html, body { 
    height: 100%;
  }
</style>

second With that in place, we can take advantage of a new feature in Bootstrap 4, which is applying h-100 to these classes to fill 100% of the space. 
We are also going to take this opportunity to add the text area and label, as well as giving them IDs that we can look up from our TypeScript code:

<div class="container-fluid h-100">
  <div class="row h-100">
    <div class="col-lg-6">
      <textarea class="form-control h-100" id="markdown"></textarea>
    </div>
    <div class="col-lg-6 h-100">
      <label class="h-100" id="markdown-output"></label>
    </div>
  </div>
</div>

we are going to add a file called MarkdownParser.ts to hold our TypeScript code and add the following code to it:

class HtmlHandler {
    public TextChangeHandler(id : string, output : string) : void {
        let markdown = <HTMLTextAreaElement>document.getElementById(id);
        let markdownOutput = <HTMLLabelElement>document.getElementById(output);
        if (markdown !== null) {
            markdown.onkeyup = (e) => {
                if (markdown.value) {
                    markdownOutput.innerHTML = markdown.value;
                }
                else 
                   markdownOutput.innerHTML = "<p></p>";
            }
        }
    }
}

We created this class so that we could get the text area and the label based on their IDs. 
Once we have these, we are going to hook into the text area, key up the event, and write the keypress value back to the label

TypeScript implicitly gives us access to standard web page behaviors. This allows us to retrieve the text area and label based on the IDs we previously entered, and to cast them to the appropriate type.
With this, we gain the ability to do things such as subscribe to events or access an element's innerHTML.

Just before the </body> tag, we add  the following to reference the JavaScript file that TypeScript produces, in order to create an instance of our HtmlHandler class 
and hook the markdown and markdown-output elements together:

<script src="script/MarkdownParser.js">
</script>
<script>
  new HtmlHandler().TextChangeHandler("markdown", "markdown-output");
</script>

so after that our html file will look like this

<!doctype html>
<html lang="en">
 <head>
 <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
 <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css" integrity="sha384-MCw98/SFnGE8fJT3GXwEOngsV7Zt27NXFoaoApmYm81iuXoPkFOJwJ8ERdknLPMO" crossorigin="anonymous">
 <style>
 html, body { 
 height: 100%; 
 }
 </style>
 <title>Advanced TypeScript - Chapter 2</title>
 </head>
 <body>
 <div class="container-fluid h-100">
 <div class="row h-100">
 <div class="col-lg-6">
 <textarea class="form-control h-100" id="markdown"></textarea>
 </div>
 <div class="col-lg-6 h-100">
 <label class="h-100" id="markdown-output"></label>
 </div>
 </div>
 </div>
 <script src="https://code.jquery.com/jquery-3.3.1.slim.min.js" integrity="sha384-q8i/X+965DzO0rT7abK41JStQIAqVgRVzpbzo5smXKp4YfRvH+8abtTE1Pi6jizo" crossorigin="anonymous"></script>
 <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.3/umd/popper.min.js" integrity="sha384-ZMP7rVo3mIykV+2+9J3UJ46jBk0WLaUAdn689aCwoqbBJiSnjAK/l8WvCWPIPm49" crossorigin="anonymous"></script>
 <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/js/bootstrap.min.js" integrity="sha384-ChfqqxuZUCnJSK3+MXmPNIyE6ZbWh2IMqE241rYiqJxyMiZ6OW/JmZQ5stwEULTy" crossorigin="anonymous"></script>

 <script src="script/MarkdownParser.js">
 </script>
 <script>
 new HtmlHandler().TextChangeHandler("markdown", "markdown-output");
 </script>
 </body>
</html>

                           **************Mapping our markdown tag types to HTML tag types*******************

In our project overview , we set out a master list of tags that our parser is going to handle. In order to identify these tags, we are going to add an enumeration consisting of the tags we are making available to our users:

enum TagType {
    Paragraph,
    Header1,
    Header2,
    Header3,
    HorizontalRule
}
we also know that we need to translate between these tags and their equivalent opening and closing HTML tags. The way that we are going to do this is to map tagType to an equivalent HTML tag. 
To do this, we are going to create a class that has the sole responsibility of handling this mapping for us. The following code shows this:

class TagTypeToHtml {
    private readonly tagType : Map<TagType, string> = new Map<TagType, string>();
    constructor() {
        this.tagType.set(TagType.Header1, "h1");
        this.tagType.set(TagType.Header2, "h2");
        this.tagType.set(TagType.Header3, "h3");
        this.tagType.set(TagType.Paragraph, "p");
        this.tagType.set(TagType.HorizontalRule, "hr")
    }
}

At first, the use of readonly on a type can appear confusing. What this keyword means is that, after the class has been instantiated, tagType cannot be recreated elsewhere in the class. This means that we can set up our mappings in the constructor safe, 
knowing that we are not going to call this.tagType = new Map<TagType, string>(); later on.
We also need a way to retrieve opening and closing tags from this class. We're going to start by creating a method to get the opening tag from tagType, as follows:

public OpeningTag(tagType : TagType) : string {
    let tag = this.tagType.get(tagType);
    if (tag !== null) {
        return `<${tag}>`;
    }
    return `<p>`;
}

This method is pretty straightforward. It starts by trying to get tagType from the map. 
With the code we currently have, we will always have an entry in the map, but we could extend the enumeration in the future and forget to add the tag to the list of tags. 
That is why we check to see if the tag is present; if it is, we return the tag enclosed in <>. If the tag is not present, we return a paragraph tag as a default.

Now, let's look at ClosingTag:

public ClosingTag(tagType : TagType) : string {
    let tag = this.tagType.get(tagType);
    if (tag !== null) {
        return `</${tag}>`;
    }
    return `</p>`;
}

Looking at these two methods, we can see that they are almost identical. 
When we think about the problem of creating our HTML tag, we realize that the only difference between an opening and a closing tag is that the closing tag has a / in it.
 With that in mind, we can change the code to use a helper method that accepts whether the tag starts with < or </:

private GetTag(tagType : TagType, openingTagPattern : string) : string {
    let tag = this.tagType.get(tagType);
    if (tag !== null) {
        return `${openingTagPattern}${tag}>`;
    }
    return `${openingTagPattern}p>`;
}

All that remains is for us to add methods to retrieve the opening and closing tags:

public OpeningTag(tagType : TagType) : string {
    return this.GetTag(tagType, `<`);
}

public ClosingTag(tagType : TagType) : string {
    return this.GetTag(tagType, `</`);
}

Pulling this all together, the code for our TagTypeToHtml class now looks like this:

class TagTypeToHtml {
    private readonly tagType : Map<TagType, string> = new Map<TagType, string>();
    constructor() {
        this.tagType.set(TagType.Header1, "h1");
        this.tagType.set(TagType.Header2, "h2");
        this.tagType.set(TagType.Header3, "h3");
        this.tagType.set(TagType.Paragraph, "p");
        this.tagType.set(TagType.HorizontalRule, "hr")
    }

    public OpeningTag(tagType : TagType) : string {
        return this.GetTag(tagType, `<`);
    }

    public ClosingTag(tagType : TagType) : string {
        return this.GetTag(tagType, `</`);
    }

    private GetTag(tagType : TagType, openingTagPattern : string) : string {
        let tag = this.tagType.get(tagType);
        if (tag !== null) {
            return `${openingTagPattern}${tag}>`;
        }
        return `${openingTagPattern}p>`;
    }
}

The single responsibility of our TagTypeToHtml class is mapping tagType to an HTML tag. 
Something that we are going to keep in object oriented  is that we want classes to have a single responsibility.



                              ******************Representing our converted markdown using a markdown document************************

While we are parsing our content, we need a mechanism to actually store the text that we are creating during the parsing process. 
We could just use a global string and update it directly, but that would become problematic if we decided to asynchronously add to it later on. 
The main reason for not using a string is down to Single Responsibility Principle again. If we were using a simple string, then each piece of code that we add to the text would end up having to write to the string in the correct way, which means that they would be mixing reading the markdown with writing to the HTML output.
 When we discuss it like that, it becomes apparent that we need to have a separate means of writing the HTML content out.What this means for us is that we are going to want code that can accept a number of strings to form the content.

We also want a means of getting our document when we have finished building it up. We are going to start by defining an interface, which will act as the contract that consuming code will implement. 
Of particular interest here is that we are going to allow our code to accept any number of items in our Add method, so we will be using a REST parameter here:

interface IMarkdownDocument {
    Add(...content : string[]) : void;
    Get() : string;
}

Given this interface, we can create our MarkdownDocument class as follows:


class MarkdownDocument implements IMarkdownDocument {
    private content : string = "";
    Add(...content: string[]): void {
        content.forEach(element => {
            this.content += element;
        });
    } 
    Get(): string {
        return this.content;
    }
}


This class is incredibly straightforward. For each piece of content passed in to our Add method, we add it to a member variable called content. 
As this is declared as private, our Get method returns the same variable. This is why I like having classes with a single responsibility—in this case, 
they are just updating the content; they tend to be a lot cleaner and easier to understand than convoluted classes that do many different things. 
The main thing is that we can do whatever we like to keep our content updated internally, as we have hidden how we maintain the document from the consuming code.

As we are going to be parsing our document one line at a time, we are going to use a class to represent the current line that we are processing:

class ParseElement {
    CurrentLine : string = "";
}


                                                 ******************Understanding the visitor pattern***********************

One of the motivations behind us using the visitor pattern is that we want to take the common ParseElement class and apply different operations on it, depending on what the underlying markdown is, 
which ultimately leads to us building up the MarkdownDocument class. The idea here is that if the content the user types in is something we would represent in HTML as a paragraph, we want to add different tags to those used, 
for example, when the content represents a horizontal rule. The convention for the visitor pattern is that we have two interfaces, IVisitor and IVisitable. At their most basic, these interfaces look like this:


interface IVisitor {
    Visit(......);
}
interface IVisitable {
    Accept(IVisitor, .....);
}

The idea behind these interfaces is that the object will be visitable, so when it needs to perform the relevant operations, it accepts the visitor so that it can visit the object.



                                            **************************Applying the visitor pattern to our code*****************

First, we are going to create the IVisitor and IVisitable interfaces as follows:

interface IVisitor {
    Visit(token : ParseElement, markdownDocument : IMarkdownDocument) : void;
}
interface IVisitable {
    Accept(visitor : IVisitor, token : ParseElement, markdownDocument : IMarkdownDocument) : void;
}

When our code reaches the point where Visit is called, we are going to use the TagTypeToHtml class to add the relevant opening HTML tag, the line of text, and then the matching closing HTML tag to our MarkdownDocument. 
As this is common to each of our tag types, we can implement a base class that encapsulates this behavior, as follows:

abstract class VisitorBase implements IVisitor {
    constructor (private readonly tagType : TagType, private readonly TagTypeToHtml : TagTypeToHtml) {}
    Visit(token: ParseElement, markdownDocument: IMarkdownDocument): void {
        markdownDocument.Add(this.TagTypeToHtml.OpeningTag(this.tagType), token.CurrentLine, 
            this.TagTypeToHtml.ClosingTag(this.tagType));
    }
}

Next, we need to add the concrete visitor implementations. This is as simple as creating the following classes:


class Header1Visitor extends VisitorBase {
    constructor() {
        super(TagType.Header1, new TagTypeToHtml());
    }
}
class Header2Visitor extends VisitorBase {
    constructor() {
        super(TagType.Header2, new TagTypeToHtml());
    }
}
class Header3Visitor extends VisitorBase {
    constructor() {
        super(TagType.Header3, new TagTypeToHtml());
    }
}
class ParagraphVisitor extends VisitorBase {
    constructor() {
        super(TagType.Paragraph, new TagTypeToHtml());
    }
}
class HorizontalRuleVisitor extends VisitorBase {
    constructor() {
        super(TagType.HorizontalRule, new TagTypeToHtml());
    }
}

At first, this code may seem like overkill, but it serves a purpose. If we take Header1Visitor, for instance, we have a class that has the single responsibility of taking the current line and adding it to our markdown document wrapped in H1 tags.

The other side of the visitor pattern code is the IVisitable implementation. For our current code, we know that we want to visit the relevant visitor whenever we call Accept. 
What this means to our code is that we can have a single visitable class that implements our IVisitable interface. This is shown in the following code:

class Visitable implements IVisitable {
    Accept(visitor: IVisitor, token: ParseElement, markdownDocument: IMarkdownDocument): void {
        visitor.Visit(token, markdownDocument);
    }
}

If we start off with our base class, we can see what this pattern gives us and how we are going to use it:

abstract class Handler<T> {
    protected next : Handler<T> | null = null;
    public SetNext(next : Handler<T>) : void {
        this.next = next;
    }
    public HandleRequest(request : T) : void {
        if (!this.CanHandle(request)) {
            if (this.next !== null) {
                this.next.HandleRequest(request);
            }
            return;
        }
    }
    protected abstract CanHandle(request : T) : boolean;
}

The next class in our chain is set using SetNext. HandleRequest works by calling our abstract CanHandle method to see whether the current class can handle the request. 
If it cannot handle the request and if this.next is not null (note the use of union types here), we forward the request onto the next class. 
This is repeated until we can either handle the request or this.next is null.

We can now add a concrete implementation of our Handler class. First, we will add our constructor and member variables, as follows:

class ParseChainHandler extends Handler<ParseElement> {
    private readonly visitable : IVisitable = new Visitable();
    constructor(private readonly document : IMarkdownDocument, 
        private readonly tagType : string, 
        private readonly visitor : IVisitor) {
        super();
    }
}

Our constructor accepts the instance of the markdown document; the string that represents our tagType, for example, #; and the relevant visitor will visit the class if we get a matching tag. 
Before we see what the code for CanHandle looks like, we need to take a slight detour and introduce a class that will help us parse the current line and see if the tag is present at the start.

We are going to create a class that exists purely to parse the string, and looks to see if it starts with the relevant markdown tag. 
What is special about our Parse method is that we are returning something called a tuple. We can think of a tuple as a fixed-size array that can have different types at different positions in the array. 
In our case, we are going to return a boolean type and a string type. The boolean type indicates whether or not the tag was found, and the string type will return the text without the tag at the start.
 for example, if the string was # Hello and the tag was # , we would want to return Hello. 
The code that checks for the tag is very straightforward; it simply looks to see if the text starts with the tag. If it does, we set the boolean part of our tuple to true and use substr to get the remainder of our text.
 Consider the following code:
class LineParser {
    public Parse(value : string, tag : string) : [boolean, string] {
        let output : [boolean, string] = [false, ""];
        output[1] = value;
        if (value === "") {
            return output;
        }
        let split = value.startsWith(`${tag}`);
        if (split) {
            output[0] = true;
            output[1] = value.substr(tag.length);
        }
        return output;
    }
}

Now that we have our LineParser class, we can apply that in our CanHandle method as follows:

protected CanHandle(request: ParseElement): boolean {
    let split = new LineParser().Parse(request.CurrentLine, this.tagType);
    if (split[0]){
        request.CurrentLine = split[1];
        this.visitable.Accept(this.visitor, request, this.document);
    }
    return split[0];
}
Here, we are using our parser to build a tuple where the first parameter states whether or not the tag was present, and the second parameter contains the text without the tag if the tag was present. 
If the markdown tag was present in our string, we call the Accept method on our Visitable implementation.

Here, we are using our parser to build a tuple where the first parameter states whether or not the tag was present,
 and the second parameter contains the text without the tag if the tag was present. If the markdown tag was present in our string, 
we call the Accept method on our Visitable implementation

class ParseChainHandler extends Handler<ParseElement> {
    private readonly visitable : IVisitable = new Visitable();
    protected CanHandle(request: ParseElement): boolean {
        let split = new LineParser().Parse(request.CurrentLine, this.tagType);
        if (split[0]){
            request.CurrentLine = split[1];
            this.visitable.Accept(this.visitor, request, this.document);
        }
        return split[0];
    }
    constructor(private readonly document : IMarkdownDocument, 
        private readonly tagType : string, 
        private readonly visitor : IVisitor) {
        super();
    }
}

We have a special case that we need to handle. 
We know that the paragraph has no tag associated with it—if there are no matches through the rest of the chain, by default, it's a paragraph. 
This means that we need a slightly different handler to cope with paragraphs, shown as follows:

class ParagraphHandler extends Handler<ParseElement> {
    private readonly visitable : IVisitable = new Visitable();
    private readonly visitor : IVisitor = new ParagraphVisitor()
    protected CanHandle(request: ParseElement): boolean {
        this.visitable.Accept(this.visitor, request, this.document);
        return true;
    }
    constructor(private readonly document : IMarkdownDocument) {
        super();
    }
}

With this infrastructure in place, we are now ready to create the concrete handlers for the appropriate tags as follows:

class Header1ChainHandler extends ParseChainHandler {
    constructor(document : IMarkdownDocument) {
        super(document, "# ", new Header1Visitor());
    }
}

class Header2ChainHandler extends ParseChainHandler {
    constructor(document : IMarkdownDocument) {
        super(document, "## ", new Header2Visitor());
    }
}

class Header3ChainHandler extends ParseChainHandler {
    constructor(document : IMarkdownDocument) {
        super(document, "### ", new Header3Visitor());
    }
}

class HorizontalRuleHandler extends ParseChainHandler {
    constructor(document : IMarkdownDocument) {
        super(document, "---", new HorizontalRuleVisitor());
    }
}


We now have a route through from the tag, for example, ---,to the appropriate visitor. 
We have now linked our chain-of-responsibility pattern to our visitor pattern. 
We have one final thing that we need to do: set up the chain. To do this, let's use a separate class that builds our chain:

class ChainOfResponsibilityFactory {
    Build(document : IMarkdownDocument) : ParseChainHandler {
        let header1 : Header1ChainHandler = new Header1ChainHandler(document);
        let header2 : Header2ChainHandler = new Header2ChainHandler(document);
        let header3 : Header3ChainHandler = new Header3ChainHandler(document);
        let horizontalRule : HorizontalRuleHandler = new HorizontalRuleHandler(document);
        let paragraph : ParagraphHandler = new ParagraphHandler(document);

        header1.SetNext(header2);
        header2.SetNext(header3);
        header3.SetNext(horizontalRule);
        horizontalRule.SetNext(paragraph);

        return header1;
    }
}


This simple-looking method accomplishes a lot for us. The first few statements initialize the chain-of-responsibility handlers for us; first for the headers, then for the horizontal rule, and finally for the paragraph handler.
 Remembering that this is only part of what we need to do here, we then go through the headers and the horizontal rule and set up the next item in the chain. 
Header 1 will forward calls on to header 2, header 2 forwards to header 3, and so on. The reason we don't set any further chained items after the paragraph handler is because that is the last case we want to handle. 
If the user isn't typing header1, header2, header3, or horizontalRule, then we're going to treat this as a paragraph.


The last class that we are going to write is used to take the text that the user is typing in and split it into individual lines, and create our ParseElement, chain-of-responsibility handlers, and MarkdownDocument instance. 
Each line is then forwarded to Header1ChainHandler to start the processing of the line. Finally, we get the text from the document and return it so that we can display it in the label:

class Markdown {
    public ToHtml(text : string) : string {
        let document : IMarkdownDocument = new MarkdownDocument();
        let header1 : Header1ChainHandler = new ChainOfResponsibilityFactory().Build(document);
        let lines : string[] = text.split(`\n`);
        for (let index = 0; index < lines.length; index++) {
            let parseElement : ParseElement = new ParseElement();
            parseElement.CurrentLine = lines[index];
            header1.HandleRequest(parseElement);
        }
        return document.Get();
    }
}



Now that we can generate our HTML content, we have one change left to do. We are going to revisit the HtmlHandler method and change it so that it calls our ToHtml markdown method.
At the same time, we are going to address an issue with our original implementation where refreshing the page loses our content until we press a key. To handle this, we are going to add a window.onload event handler

class HtmlHandler {
 private markdownChange : Markdown = new Markdown;
    public TextChangeHandler(id : string, output : string) : void {
        let markdown = <HTMLTextAreaElement>document.getElementById(id);
        let markdownOutput = <HTMLLabelElement>document.getElementById(output);
    
        if (markdown !== null) {
            markdown.onkeyup = (e) => {
                this.RenderHtmlContent(markdown, markdownOutput);
            }
            window.onload = (e) => {
                this.RenderHtmlContent(markdown, markdownOutput);
            }
        }
    }

    private RenderHtmlContent(markdown: HTMLTextAreaElement, markdownOutput: HTMLLabelElement) {
        if (markdown.value) {
            markdownOutput.innerHTML = this.markdownChange.ToHtml(markdown.value);
        }
        else
            markdownOutput.innerHTML = "<p></p>";
    }
}


Now, when we run our application, it displays the rendered HTML content, even when we refresh our page.
 We have successfully created a simple markdown editor that satisfies the points that we laid out in our requirements, gathering stage.
