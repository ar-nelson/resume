const { task, file, directory } = require("jake");
const { promisify } = require("util");
const fs = require("fs");
const readFile = promisify(fs.readFile);
const writeFile = promisify(fs.writeFile);
const rimraf = promisify(require("rimraf"));
const marked = require("marked");
const Mustache = require("mustache");
const sass = require("sass");
const puppeteer = require("puppeteer");
const nodemon = require("nodemon");

const PATH_DIR = "build";
const PATH_PHONE = ".phone";
const PATH_MARKDOWN = "README.md";
const PATH_SASS = "stylesheet.scss";
const PATH_TEMPLATE = "template.html";
const PATH_HTML = `${PATH_DIR}/resume.html`;
const PATH_CSS_BASE = "stylesheet.css";
const PATH_CSS = `${PATH_DIR}/${PATH_CSS_BASE}`;
const PATH_PDF = `${PATH_DIR}/resume.pdf`;

directory(PATH_DIR);

file(PATH_HTML, [PATH_DIR, PATH_MARKDOWN, PATH_TEMPLATE], async () => {
  let phone;
  try {
    phone = await readFile(PATH_PHONE, { encoding: "utf8" });
  } catch (e) {
    throw new Error(`
      Cannot read file "${PATH_PHONE}".
      The repository does not include a phone number.
      Please create a "${PATH_PHONE}" file containing a phone number.
    `);
  }
  let markdown = await readFile(PATH_MARKDOWN, { encoding: "utf8" });
  markdown = Mustache.render(markdown, { phone });
  const rawHtml = marked.parse(markdown);
  const template = await readFile(PATH_TEMPLATE, { encoding: "utf8" });
  const finalHtml = Mustache.render(template, {
    body: rawHtml,
    stylesheet: PATH_CSS_BASE,
  });
  await writeFile(PATH_HTML, finalHtml, { encoding: "utf8" });
});

file(PATH_CSS, [PATH_DIR, PATH_SASS], async () => {
  const result = sass.renderSync({ file: PATH_SASS });
  if (result.css) {
    await writeFile(PATH_CSS, result.css);
  } else {
    throw result;
  }
});

file(PATH_PDF, [PATH_HTML, PATH_CSS], async () => {
  const browser = await puppeteer.launch({ headless: true });
  const page = await browser.newPage();
  await page.goto(`file:///${__dirname}/${PATH_HTML}`, {
    waitUntil: "networkidle0",
  });
  await page.pdf({ format: "Letter", path: PATH_PDF });
  await browser.close();
});

task("default", [PATH_PDF]);

task("watch", () => {
  nodemon({
    ext: "md html scss",
    ignore: ["build"],
    exec: "jake",
    execMap: {
      sh: "/bin/sh",
      bat: "cmd.exe /c",
      cmd: "cmd.exe /c",
    },
  }).on("restart", (files) => {
    console.log("*** Restarting due to", files);
  });
});

task("clean", [], async () => {
  await rimraf(PATH_DIR);
});
