# Hello, Square!

WIP

## Creating and Opening the Screen Class

- Create a class like this
- < a basic facade screen with a red square in the middle >
```java
~import com.teamwizardry.librarianlib.facade.FacadeScreen;
~import com.teamwizardry.librarianlib.facade.layers.RectLayer;
~import com.teamwizardry.librarianlib.math.Vec2d;
~import net.minecraft.util.text.TranslationTextComponent;
~
~import java.awt.Color;
~
public class HelloSquareScreen extends FacadeScreen {
    public HelloSquareScreen() {
        super(new TranslationTextComponent("modid.screen.hello_square.title"));

        getMain().setSize(new Vec2d(20, 20));

        RectLayer redSquare = new RectLayer(Color.RED, 0, 0, 20, 20);
        getMain().add(redSquare);
    }
}
```
- Open the screen like this (NOTE: CLIENT ONLY!)
- You should see this:

![](hello_square.png)

## Anatomy of a Facade Screen

- The screen
  - The title
  - The setup code
- Layers: what are they, briefly.
  - The main layer
  - The square layer
