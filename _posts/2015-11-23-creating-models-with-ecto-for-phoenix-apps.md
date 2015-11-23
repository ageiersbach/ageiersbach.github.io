Creating models with [Ecto]() was surprisingly easy.

On my Phoenix app, [Inferno](github.com/ageiersbach/inferno), I had already created a notes resource, controller, and view, so I was prepared for conflict when I used `mix phoenix.gen.html` to generate the model. I was pleasantly surprised at how smoothly it was handled.

```
mix phoenix.gen.html Note notes title:string body:text sort_order:integer
Compiled web/views/flames_view.ex
Compiled web/views/hello_view.ex
Generated inferno app
* creating priv/repo/migrations/20151123152131_create_note.exs
* creating web/models/note.ex
* creating test/models/note_test.exs
* creating web/controllers/note_controller.ex
web/controllers/note_controller.ex already exists, overwrite? [Yn] Y
* creating web/templates/note/edit.html.eex
* creating web/templates/note/form.html.eex
* creating web/templates/note/index.html.eex
web/templates/note/index.html.eex already exists, overwrite? [Yn] Y
* creating web/templates/note/new.html.eex
* creating web/templates/note/show.html.eex
* creating web/views/note_view.ex
web/views/note_view.ex already exists, overwrite? [Yn] Y
* creating test/controllers/note_controller_test.exs
 ```
 
 I allowed the overwriting of the controller and views, since I didn't really care to preserve what I had already. 
 
 
