# Src-parser.js
// AdaptML Parser v1

class Parser {
  constructor(tokens) {
    this.tokens = tokens;
    this.position = 0;
  }

  current() {
    return this.tokens[this.position];
  }

  advance() {
    return this.tokens[this.position++];
  }

  expect(type) {
    const token = this.current();

    if (token.type !== type) {
      throw new Error(
        `Expected ${type}, got ${token.type}`
      );
    }

    return this.advance();
  }

  parse() {
    const children = [];

    while (this.current().type !== "EOF") {
      children.push(this.parseElement());
    }

    return {
      type: "Document",
      children
    };
  }

  parseElement() {
    this.expect("OPEN_TAG");

    const name = this.expect("NAME").value;

    const attributes = {};

    // Attributes
    while (
      this.current().type === "NAME"
    ) {
      const attrName =
        this.advance().value;

      this.expect("EQUALS");

      const attrValue =
        this.expect("STRING").value;

      attributes[attrName] = attrValue;
    }

    // Self closing tag
    if (this.current().type === "SLASH") {
      this.advance();
      this.expect("CLOSE_TAG");

      return {
        type: "Element",
        name,
        attributes,
        children: []
      };
    }

    this.expect("CLOSE_TAG");

    const children = [];

    // Children
    while (
      this.current().type !== "EOF"
    ) {
      if (
        this.current().type === "OPEN_TAG" &&
        this.tokens[this.position + 1]?.type === "SLASH"
      ) {
        break;
      }

      children.push(
        this.parseElement()
      );
    }

    // Closing tag
    this.expect("OPEN_TAG");
    this.expect("SLASH");

    const closingName =
      this.expect("NAME").value;

    this.expect("CLOSE_TAG");

    if (closingName !== name) {
      throw new Error(
        `Closing tag </${closingName}> does not match <${name}>`
      );
    }

    return {
      type: "Element",
      name,
      attributes,
      children
    };
  }
}

module.exports = Parser;
