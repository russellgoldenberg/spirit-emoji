<script>
  import * as d3 from "d3";
  import emojiData from "./emojis";
  export let title;
  let prev = null;
  let emoji = null;
  let name = null;
  let visible = false;
  let includeFlags = false;

  function flagCheck(d) {
    if (includeFlags) return true;
    return !d.shortname.includes("flag:");
  }

  function getEmoji(i) {
    if (!i.length) return null;

    const chars = [...i];
    const exp = chars.map(d => `.*${d}`).join("");
    const reg = new RegExp(exp);
    const filtered = emojiData.filter(
      d => flagCheck(d) && reg.test(d.shortname)
    );

    if (!filtered.length) return null;

    const withLength = filtered.map(d => {
      const { shortname } = d;
      const [match] = shortname.match(reg);
      const start = shortname.indexOf(chars[0]);
      const end = match.length;
      const sub = shortname.substring(start, end);
      const len = sub.length;
      return { ...d, len };
    });

    withLength.sort((a, b) => d3.descending(a.len, b.len)) ||
      d3.descending(+a.number, +b.number);

    const top = withLength.pop();
    const start = { str: top.shortname, index: 0 };
    top.marked = chars.reduce((prev, cur) => {
      const pre = prev.str.substring(0, prev.index);
      const after = prev.str.substring(prev.index, prev.str.length);
      const end = after.indexOf(cur);
      const inject = after.substring(0, end);
      const post = after.substring(end + 1);
      console.log({ pre, after, end, inject, post });

      const str = `${pre}${inject}<mark>${cur}</mark>${post}`;
      const index = str.length - post.length;
      return { str, index };
    }, start).str;

    return top;
  }

  function handleInput() {
    const input = this.value.replace(/\W/g, "");
    if (!input.length) prev = null;

    const output = getEmoji(input) || prev;

    if (output) {
      emoji = output.browser;
      name = output.marked;
      prev = output;
    }

    visible = !!output ? "is-visible" : "";
  }
</script>

<style>
  main {
    max-width: 60rem;
    margin: 0 auto;
    padding: 1rem;
  }

  h1 {
    font-size: 2em;
    line-height: 1.2;
    margin: 0;
    text-align: center;
  }

  input {
    font-size: 1.25em;
    display: block;
    width: 90%;
    margin: 2rem auto;
    padding: 0.25em;
    max-width: 30rem;
  }

  div.output {
    visibility: hidden;
  }

  p.emoji {
    font-size: 4em;
    margin: 0;
    text-align: center;
    line-height: 1;
  }

  p.name {
    font-size: 1.25em;
    margin: 0;
    text-align: center;
    color: #999;
  }

  div.is-visible {
    visibility: visible;
  }

  @media (min-width: 640px) {
    h1 {
      font-size: 3em;
    }

    p,
    input {
      font-size: 2em;
    }

    p.emoji {
      font-size: 8em;
    }
  }
</style>

<main>
  <h1>{title}</h1>
  <input on:input={handleInput} placeholder="Enter your name" />
  <div class="output {visible}">
    <p class="emoji">{emoji}</p>
    <p class="name">
      {@html name}
    </p>
  </div>
</main>