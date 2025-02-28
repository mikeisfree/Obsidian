---
title: Learn how to create an organic distortion effect for text using JavaScript and CSS for a unique, fluid animation.
draft: false
tags: 
vault: mike
---
 

![[Pasted image 20250224145341.png]]


https://tympanus.net/Tutorials/OrganicTextDistortion
https://github.com/JorgeCapillo/infinite-scrolling-text-distortion



Hi, everyone! It’s me again, [Jorge Toloza](https://jorgetoloza.co/) 👋 Last time I showed you [how to code a progressive blur effect](https://tympanus.net/codrops/2024/07/02/progressive-blur-effect-using-webgl-with-ogl-and-glsl-shaders/) and in today’s tutorial, we’ll create a dynamic text distortion effect that responds to scrolling, giving text an organic, wave-like motion.

Using JavaScript and CSS, we’ll apply a sine wave to text in the left column and a cosine wave to the right, creating unique movements that adapt to scroll speed and direction. This interactive effect brings a smooth, fluid feel to typography, making it an interesting visual effect.

Let’s get started!

## The HTML Markup

The structure is very basic, consisting of only a `<div>` containing our content, which is separated by `<p>` tags.

```markup
<div class="column">
	<div class="column-content">
		<p>Once completing the screenplay with Filippou and during the production process...</p>
	</div>
</div>
```

CSS Styles
Let’s add some styles for our columns, including width, paragraph margins, and drop-cap styling for the first letter. In JavaScript, we’ll also be splitting our paragraphs into individual words.

<script>
```
:root {
  --margin: 40rem;
  --gap: 20rem;
  --column: calc((var(--rvw) * 100 - var(--margin) * 2 - var(--gap) * 9) / 10);
}
.column {
  width: calc(var(--column) * 2 + var(--gap));
  height: 100%;
  flex-shrink: 0;
  p {
    margin: 1em 0;
  }
  p.drop-cap {
    .line:first-child {
      .word:first-child {
        height: 1em;
        &::first-letter {
          font-size: 6.85500em;
        }
      }
    }
  }
}
```
</script>

Splitting the Text
The plan is to move each line independently to create an organic movement effect. To achieve this, we need to separate each paragraph into lines. The logic is straightforward: we split the text into words and start adding them to the first line. If the line becomes wider than the maximum width, we create a new line and add the word there, continuing this process for the entire paragraph.

This is the Utils file


<script>
```
const lineBreak = (text, max, $container) => {
  const getTotalWidth = ($el) =>
    Array.from($el.children).reduce((acc, child) => acc + child.getBoundingClientRect().width, 0);
  
  const createNewLine = () => {
    const $line = document.createElement('span');
    $line.classList.add('line');
    return $line;
  };
  // Step 1: Break text into words and wrap in span elements
  const words = text.split(/\s/).map((w, i) => {
    const span = document.createElement('span');
    span.classList.add('word');
    span.innerHTML = (i > 0 ? ' ' : '') + w;
    return span;
  });

  // Step 2: Insert words into the container
  $container.innerHTML = '';
  words.forEach(word => $container.appendChild(word));

  // Step 3: Add left-space and right-space classes
  words.forEach((word, i) => {
    if (i > 0 && word.innerHTML.startsWith(' ')) {
      word.classList.add('left-space');
    }
    if (word.innerHTML.endsWith(' ')) {
      word.classList.add('right-space');
    }
  });

  // Step 4: Calculate total width and create new lines if necessary
  if (getTotalWidth($container) > max) {
    $container.innerHTML = '';
    let $currentLine = createNewLine();
    $container.appendChild($currentLine);

    words.forEach(word => {
      $currentLine.appendChild(word);
      if (getTotalWidth($currentLine) > max) {
        $currentLine.removeChild(word);
        $currentLine = createNewLine();
        $currentLine.appendChild(word);
        $container.appendChild($currentLine);
      }
    });
  } else {
    // If no line break is needed, just put all words in a single line
    const $line = createNewLine();
    words.forEach(word => $line.appendChild(word));
    $container.appendChild($line);
  }

  // Step 5: Wrap lines in `.text` span and remove empty lines
  Array.from($container.querySelectorAll('.line')).forEach(line => {
    if (line.innerText.trim()) {
      line.innerHTML = `<span class="text">${line.innerHTML}</span>`;
    } else {
      line.remove();
    }
  });
};

const getStyleNumber = (el, property) => {
  return Number(getComputedStyle(el)[property].replace('px', ''));
}
const isTouch = () => {
  try {
    document.createEvent('TouchEvent');
    return true;
  } catch (e) {
    return false;
  }
}

</script> 
```




With our utility functions ready, let’s work on the SmartText component. It will parse our HTML to prepare it for use in our lineBreak function.





```
<script>

import Util from '../util/util.js';

export default class SmarText {
  constructor(options) {
    this.$el = options.$el;
    this.text = this.$el.innerText;
    this.init();
  }

  init() {
    // Parse words from the element's content
    this.words = this.parseWords();
    this.$el.innerHTML = '';

    // Convert each word to a separate HTML element and append to container
    this.words.forEach((word) => {
      const element = this.createWordElement(word);
      this.$el.appendChild(element);
    });

    // Apply line breaks to achieve responsive layout
    this.applyLineBreaks();
  }

  // Parse words from <p> and header elements, distinguishing text and anchor links
  parseWords() {
    const words = [];
    this.$el.querySelectorAll('p, h1, h2, h3, h4, h5, h6').forEach((p) => {
      p.childNodes.forEach((child) => {
        if (child.nodeType === 3) { // If text node
          const text = child.textContent.trim();
          if (text !== '') {
            // Split text into words and wrap each in a SPAN element
            words.push(...text.split(' ').map((w) => ({ type: 'SPAN', word: w })));
          }
        } else if (child.tagName === 'A') { // If anchor link
          const text = child.textContent.trim();
          if (text !== '') {
            // Preserve link attributes (href, target) for word element
            words.push({ type: 'A', word: text, href: child.href, target: child.target });
          }
        } else {
          // For other element types, recursively parse child nodes
          words.push(...this.parseChildWords(child));
        }
      });
    });
    return words;
  }

  // Recursive parsing of child elements to handle nested text
  parseChildWords(node) {
    const words = [];
    node.childNodes.forEach((child) => {
      if (child.nodeType === 3) { // If text node
        const text = child.textContent.trim();
        if (text !== '') {
          // Split text into words and associate with the parent tag type
          words.push(...text.split(' ').map((w) => ({ type: node.tagName, word: w })));
        }
      }
    });
    return words;
  }

  // Create an HTML element for each word, with classes and attributes as needed
  createWordElement(word) {
    const element = document.createElement(word.type);
    element.innerText = word.word;
    element.classList.add('word');

    // For anchor links, preserve href and target attributes
    if (word.type === 'A') {
      element.href = word.href;
      element.target = word.target;
    }
    return element;
  }

  // Apply line breaks based on the available width, using Util helper functions
  applyLineBreaks() {
    const maxWidth = Util.getStyleNumber(this.$el, 'maxWidth');
    const parentWidth = this.$el.parentElement?.clientWidth ?? window.innerWidth;

    // Set the final width, limiting it by maxWidth if defined
    let finalWidth = 0;
    if (isNaN(maxWidth)) {
      finalWidth = parentWidth;
    } else {
      finalWidth = Math.min(maxWidth, parentWidth);
    }

    // Perform line breaking within the specified width
    Util.lineBreak(this.text, finalWidth, this.$el);
  }
}
```
</script>


We’re essentially placing the words in an array, using <a> tags for links and <span> tags for the rest. Then, we split the text into lines each time the user resizes the window.

The Column Class
Now for our Column class: we’ll move each paragraph with the wheel event to simulate scrolling. On resize, we split each paragraph into lines using our SmartText class. We then ensure there’s enough content to create the illusion of infinite scrolling; if not, we clone a few paragraphs.
 
<script>
// Importing the SmartText component, likely for advanced text manipulation.
import SmartText from './smart-text.js';

export default class Column {
  constructor(options) {
    // Set up main element and configuration options.
    this.$el = options.el;
    this.reverse = options.reverse;

    // Initial scroll parameters to control smooth scrolling.
    this.scroll = {
      ease: 0.05,     // Ease factor for smooth scrolling effect.
      current: 0,     // Current scroll position.
      target: 0,      // Desired scroll position.
      last: 0         // Last recorded scroll position.
    };

    // Tracking touch states for touch-based scrolling.
    this.touch = {prev: 0, start: 0};

    // Speed control, defaulting to 0.5.
    this.speed = {t: 1, c: 1};
    this.defaultSpeed = 0.5;

    this.target = 0;  // Target position for animations.
    this.height = 0;  // Total height of content.
    this.direction = ''; // Track scrolling direction.
    
    // Select main content area and paragraphs inside it.
    this.$content = this.$el.querySelector('.column-content');
    this.$paragraphs = Array.from(this.$content.querySelectorAll('p'));

    // Bind event handlers to the current instance.
    this.resize = this.resize.bind(this);
    this.render = this.render.bind(this);
    this.wheel = this.wheel.bind(this);
    this.touchend = this.touchend.bind(this);
    this.touchmove = this.touchmove.bind(this);
    this.touchstart = this.touchstart.bind(this);
    
    // Initialize listeners and render loop.
    this.init();
  }

  init() {
    // Attach event listeners for window resize and scrolling.
    window.addEventListener('resize', this.resize);
    window.addEventListener('wheel', this.wheel);
    document.addEventListener('touchend', this.touchend);
    document.addEventListener('touchmove', this.touchmove);
    document.addEventListener('touchstart', this.touchstart);

    // Initial sizing and rendering.
    this.resize();
    this.render();
  }

  wheel(e) {
    // Handle scroll input using mouse wheel.
    let t = e.wheelDeltaY || -1 * e.deltaY;
    t *= .254;
    this.scroll.target += -t;
  }

  touchstart(e) {
    // Record initial touch position.
    this.touch.prev = this.scroll.current;
    this.touch.start = e.touches[0].clientY;
  }

  touchend(e) {
    // Reset target after touch ends.
    this.target = 0;
  }

  touchmove(e) {
    // Calculate scroll distance from touch movement.
    const x = e.touches ? e.touches[0].clientY : e.clientY;
    const distance = (this.touch.start - x) * 2;
    this.scroll.target = this.touch.prev + distance;
  }

  splitText() {
    // Split text into elements for individual animations.
    
    this.splits = [];
    const paragraphs = Array.from(this.$content.querySelectorAll('p'));
    paragraphs.forEach((item) => {
      item.classList.add('smart-text');  // Add class for styling.
      if(Math.random() > 0.7)
        item.classList.add('drop-cap'); // Randomly add drop-cap effect.
      this.splits.push(new SmartText({$el: item}));
    });
  }

  updateChilds() {
    // Dynamically add content copies if content height is smaller than window.
    const h = this.$content.scrollHeight;
    const ratio = h / this.winH;
    if(ratio < 2) {
      const copies = Math.min(Math.ceil(this.winH / h), 100);
      for(let i = 0; i < copies; i++) {
        Array.from(this.$content.children).forEach((item) => {
          const clone = item.cloneNode(true);
          this.$content.appendChild(clone);
        });
      }
    }
  }

  resize() {
    // Update dimensions on resize and reinitialize content.
    this.winW = window.innerWidth;
    this.winH = window.innerHeight;

    if(this.destroyed) return;
    this.$content.innerHTML = '';
    this.$paragraphs.forEach((item) => {
      const clone = item.cloneNode(true);
      this.$content.appendChild(clone);
    });
    this.splitText();     // Reapply text splitting.
    this.updateChilds();  // Ensure sufficient content for smooth scroll.

    // Reset scroll values and prepare for rendering.
    this.scroll.target = 0;
    this.scroll.current = 0;
    this.speed.t = 0;
    this.speed.c = 0;
    this.paused = true;
    this.updateElements(0);
    this.$el.classList.add('no-transform');
    
    // Initialize items with position and bounds.
    this.items = Array.from(this.$content.children).map((item, i) => {
      const data = { el: item };
      data.width = data.el.clientWidth;
      data.height = data.el.clientHeight;
      data.left = data.el.offsetLeft;
      data.top = data.el.offsetTop;
      data.bounds = data.el.getBoundingClientRect();
      data.y = 0;
      data.extra = 0;

      // Calculate line-by-line animation details.
      data.lines = Array.from(data.el.querySelectorAll('.line')).map((line, j) => {
        return {
          el: line,
          height: line.clientHeight,
          top: line.offsetTop,
          bounds: line.getBoundingClientRect()
        }
      });
      return data;
    });

    this.height = this.$content.scrollHeight;
    this.updateElements(0);
    this.speed.t = this.defaultSpeed;
    this.$el.classList.remove('no-transform');
    this.paused = false;
  }

  destroy() {
    // Clean up resources when destroying the instance.
    this.destroyed = true;
    this.$content.innerHTML = '';
    this.$paragraphs.forEach((item) => {
      item.classList.remove('smart-text');
      item.classList.remove('drop-cap');
    });
  }

  render(t) {
    // Main render loop using requestAnimationFrame.
    if(this.destroyed) return;
    if(!this.paused) {
      if (this.start === undefined) {
        this.start = t;
      }

      const elapsed = t - this.start;
      this.speed.c += (this.speed.t - this.speed.c) * 0.05;
      this.scroll.target += this.speed.c;
      this.scroll.current += (this.scroll.target - this.scroll.current) * this.scroll.ease;
      this.delta = this.scroll.target - this.scroll.current;

      // Determine scroll direction.
      if (this.scroll.current > this.scroll.last) {
        this.direction = 'down';
        this.speed.t = this.defaultSpeed;
      } else if (this.scroll.current < this.scroll.last) {
        this.direction = 'up';
        this.speed.t = -this.defaultSpeed;
      }
      
      // Update element positions and continue rendering.
      this.updateElements(this.scroll.current, elapsed);
      this.scroll.last = this.scroll.current;
    }
    window.requestAnimationFrame(this.render);
  }

  curve(y, t = 0) {
    // Curve effect to create non-linear animations.
    t = t * 0.0007;
    if(this.reverse) 
      return Math.cos(y * Math.PI + t) * (15 + 5 * this.delta / 100);
    return Math.sin(y * Math.PI + t) * (15 + 5 * this.delta / 100);
  }

  updateElements(scroll, t) {
    // Position and animate each item based on scroll position.
    if (this.items && this.items.length > 0) {
      const isReverse = this.reverse;
      this.items.forEach((item, j) => {
        // Track if items are out of viewport.
        item.isBefore = item.y + item.bounds.top > this.winH;
        item.isAfter = item.y + item.bounds.top + item.bounds.height < 0;

        if(!isReverse) {
          if (this.direction === 'up' && item.isBefore) {
            item.extra -= this.height;
            item.isBefore = false;
            item.isAfter = false;
          }
          if (this.direction === 'down' && item.isAfter) {
            item.extra += this.height;
            item.isBefore = false;
            item.isAfter = false;
          }
          item.y = -scroll + item.extra;
        } else {
          if (this.direction === 'down' && item.isBefore) {
            item.extra -= this.height;
            item.isBefore = false;
            item.isAfter = false;
          }
          if (this.direction === 'up' && item.isAfter) {
            item.extra += this.height;
            item.isBefore = false;
            item.isAfter = false;
          }
          item.y = scroll + item.extra;
        }

        // Animate individual lines within each item.
        item.lines.forEach((line, k) => {
          const posY = line.top + item.y;
          const progress = Math.min(Math.max(0, posY / this.winH), 1);
          const x = this.curve(progress, t);
          line.el.style.transform = `translateX(${x}px)`;
        });
        
        item.el.style.transform = `translateY(${item.y}px)`;
      });
    }
  }
}
Let’s take a closer look at the resize method.

resize() {
  ...

  // Reset scroll values and prepare for rendering.
  this.scroll.target = 0;
  this.scroll.current = 0;
  this.speed.t = 0;
  this.speed.c = 0;
  this.paused = true;
  this.updateElements(0);
  this.$el.classList.add('no-transform');
  
  // Initialize items with position and bounds.
  this.items = Array.from(this.$content.children).map((item, i) => {
    const data = { el: item };
    data.width = data.el.clientWidth;
    data.height = data.el.clientHeight;
    data.left = data.el.offsetLeft;
    data.top = data.el.offsetTop;
    data.bounds = data.el.getBoundingClientRect();
    data.y = 0;
    data.extra = 0;

    // Calculate line-by-line animation details.
    data.lines = Array.from(data.el.querySelectorAll('.line')).map((line, j) => {
      return {
        el: line,
        height: line.clientHeight,
        top: line.offsetTop,
        bounds: line.getBoundingClientRect()
      }
    });
    return data;
  });

  this.height = this.$content.scrollHeight;
  this.updateElements(0);
  this.speed.t = this.defaultSpeed;
  this.$el.classList.remove('no-transform');
  this.paused = false;
}
 </script>

We reset all scroll values and add a class to remove transformations so we can measure everything without offsets. Next, we set up our paragraphs, saving their bounds and variables (y and extra) for movement.

Then, for each paragraph’s lines, we save their dimensions to enable horizontal movement.

Finally, we get the full height of the column and restart the animation.

The render method is straightforward: we update all scroll variables and time. After that, we determine the current direction to move the elements accordingly. Once all variables are up to date, we call the updateElements function to start the flow.


<script>
```
`render(t) {`
  `// Main render loop using requestAnimationFrame.`
  `if(this.destroyed) return;`
  `if(!this.paused) {`
    `if (this.start === undefined) {`
      `this.start = t;`
    `}`

    `const elapsed = t - this.start;`
    `this.speed.c += (this.speed.t - this.speed.c) * 0.05;`
    `this.scroll.target += this.speed.c;`
    `this.scroll.current += (this.scroll.target - this.scroll.current) * this.scroll.ease;`
    `this.delta = this.scroll.target - this.scroll.current;`

    `// Determine scroll direction.`
    `if (this.scroll.current > this.scroll.last) {`
      `this.direction = 'down';`
      `this.speed.t = this.defaultSpeed;`
    `} else if (this.scroll.current < this.scroll.last) {`
      `this.direction = 'up';`
      `this.speed.t = -this.defaultSpeed;`
    `}`
    
    `// Update element positions and continue rendering.`
    `this.updateElements(this.scroll.current, elapsed);`
    `this.scroll.last = this.scroll.current;`
  `}`
  `window.requestAnimationFrame(this.render);`
}
```
</script>

With all that set, we can start moving things. 


<script>
```
`updateElements(scroll, t) {`
  `// Position and animate each item based on scroll position.`
  `if (this.items && this.items.length > 0) {`
    `const isReverse = this.reverse;`
    `this.items.forEach((item, j) => {`
      `// Track if items are out of viewport.`
      `item.isBefore = item.y + item.bounds.top > this.winH;`
      `item.isAfter = item.y + item.bounds.top + item.bounds.height < 0;`

      `if(!isReverse) {`
        `if (this.direction === 'up' && item.isBefore) {`
          `item.extra -= this.height;`
          `item.isBefore = false;`
          `item.isAfter = false;`
        `}`
        `if (this.direction === 'down' && item.isAfter) {`
          `item.extra += this.height;`
          `item.isBefore = false;`
          `item.isAfter = false;`
        `}`
        `item.y = -scroll + item.extra;`
      `} else {`
        `if (this.direction === 'down' && item.isBefore) {`
          `item.extra -= this.height;`
          `item.isBefore = false;`
          `item.isAfter = false;`
        `}`
        `if (this.direction === 'up' && item.isAfter) {`
          `item.extra += this.height;`
          `item.isBefore = false;`
          `item.isAfter = false;`
        `}`
        `item.y = scroll + item.extra;`
      `}`

      `// Animate individual lines within each item.`
      `item.lines.forEach((line, k) => {`
        `const posY = line.top + item.y;`
        `const progress = Math.min(Math.max(0, posY / this.winH), 1);`
        `const x = this.curve(progress, t);`
        `line.el.style.transform = `translateX(${x}px)`;`
      `});`
      
      `item.el.style.transform = `translateY(${item.y}px)`;`
    `});`
  `}`
`}`

```
</script>



This infinite scroll approach is based on Luis Henrique Bizarro’s article. It’s quite simple: we check if elements are before the viewport; if so, we move them after the viewport. For elements positioned after the viewport, we do the reverse.

Next, we handle the horizontal movement for each line. We get the line’s position by adding its top offset to the paragraph’s position, then normalize that value by dividing it by the viewport height. This gives us a ratio to calculate the X position using our curve function.

The curve function uses Math.sin for normal columns and Math.cos for reversed columns. We apply a phase shift to the trigonometric function based on the time from requestAnimationFrame, then increase the animation’s strength according to the scroll velocity.


<script>
```
curve(y, t = 0) {
  // Curve effect to create non-linear animations.
  t = t * 0.0007;
  if(this.reverse) 
    return Math.cos(y * Math.PI + t) * (15 + 5 * this.delta / 100);
  return Math.sin(y * Math.PI + t) * (15 + 5 * this.delta / 100);
}

```
</script>




After that, you should have something like this:

https://codrops-1f606.kxcdn.com/codrops/wp-content/uploads/2024/11/column_smaller.mp4?x46859


