---
layout:     post
title:      "PowerShell, Winforms, and Events"
date:       2018-07-13 23:30:00 -0800
tags:       [powershell, windows, forms]
---
At work I've been writing a script to automate some testing, but it needed a GUI component.  Since the project is in PowerShell winforms seemed like the logical choice.  I ran into a situation where I needed to click a button to spawn an open file dialog, then get the result and put that into a textbox on the same form we launched from.  I've implemented as much as I can in classes (because I'm a good little wannabe programmer), and so ... well first look at the example below.  
  
```powershell
Add-Type -AssemblyName System.Windows.Forms

class MyForm : System.Windows.Forms.Form {
    MyForm($mystuff) {
        #Do-Stuff
        $this.Add_Load( $this.MyForm_Load )
    }

    $MyForm_Load = {
        $mlabel = [System.Windows.Forms.Label]::new()
        $mlabel.Name = "trolol"
        $mlabel.Text = "hello, world!"

        $mbutton = [System.Windows.Forms.Button]::new()
        $mbutton.Text = "click me"
        $mbutton.Location = [System.Drawing.Point]::new(100,100)
        $mbutton.Add_Click( $this.mbutton_click )

        $this.Controls.Add($mlabel)
        $this.Controls.Add($mbutton)
    }

    $mbutton_click = {
        $this.Parent.Controls["trolol"].Text = "goodbye, world."
    }
}

$foo = [MyForm]::new("test")
$foo.ShowDialog()
```
  
So in the example above we have a class (`MyForm`) which inherits from `System.Windows.Forms.Form`.  The constructor on line 4 adds a Load event handler and assigns it to the script block defined on line 9.  The `MyForm_Load` scriptblock defines a label control who's name is "trolol" and default text is "hello, world!".  A button is then defined and a Click event handler is added (defined on line 23).  
  
When the button is clicked it enters the script block in the context of the button.  What I didn't know at the time was `$this` and `$_` are implied in the event handler script block (the sending control and eventargs, respectively).  So we are able to get a reference to the button when it's clicked (`$this`) and then the form (the button's parent) by using `$this.Parent`.
  
The last trick is to address the label we want to change.  We can't use `$mlabel`, but we can get to the control by name.  `$this.Parent.Controls["trolol"]` gives us a reference to the label, and we can change its text at will.  
  
In retrospect I don't know how cool of a trick this was, but finding out `$this` and `$_` were implied was a bit of a hallelujah moment for me.  Hope this helps you too!
