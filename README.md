# rollup-env-replacer
A Rollup-env-replacer is a Rollup plugin that streamlines environment-specific configurations by automatically replacing environment variables during the JavaScript bundling process.

This retrieves environment variables from `.env ` files, which will be used with the rollup-plugin-replace plugin.

Since the plugin doesn't support empty objects, we need to ensure that if any .env file is empty, an empty object ({}) won't be passed to the plugin.

> May there are other ways such as `dotenv` package to get pair key value from each file but I try to use another way.

```js
function getEnvRecord(prev, line) {
  const result = line.split("=");
  const key = `process.env.${result[0]}`;
  // try to stringfiy the value to avoid any errors
  return { ...prev, [key]: `"${result[1]}"` };
}

function getEnvFiles() {
  const envContents = {};
  // There are sample files for env you can use fs.readDir to get all possible 'env' files in the root directory
  const envFiles = ["env", "env.development", "env.production"];

  for (const file of envFiles) {
    const filePath = path.join(process.cwd(), `.${file}`);
    if (!fs.existsSync(filePath)) continue;
    const content = fs.readFileSync(filePath, "utf-8");
    // get env contents from each file and get key value of environment variables
    envContents[file] = content
      .split(/\r?\n/)
      .map((line) => line.trim())
      .reduce(getEnvRecord, {});
  }

  return envContents;
}

// This function helps you access environment files by their exact file names, and you can tailor it to fit your project's unique needs.
function getEnvReplacer() {
  const envContents = getEnvFiles();
  const env = process.env.NODE_ENV;
  if (!env || env === "development") return envContents["env.development"];
  if (env === "production") return envContents["env.production"];
  return envContents["env"];
}
```

After that we can pass `getEnvReplacer` to replace.

```js
import replace from "rollup-plugin-replace";

const rollupConfig = [
  {
    plugins: [
      replace({ /* other replacers, */ ...getEnvReplacer() }),
    ],
  }
];
```