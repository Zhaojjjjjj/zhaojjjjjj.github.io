> Config import auto aligned
```
npm install --save-dev @trivago/prettier-plugin-sort-imports
```
```
.prettierrc

{
    "printWidth": 400,
    "tabWidth": 4,
    "useTabs": false,
    "semi": true,
    "bracketSameLine": true,
    "quoteProps": "preserve",
    "trailingComma": "all",
    "singleQuote": true,
    "importOrderSeparation": true,
    "importOrderSortSpecifiers": true,
    "plugins": ["@trivago/prettier-plugin-sort-imports"],
    "importOrder": ["^@core/(.*)$", "<THIRD_PARTY_MODULES>", "^@ui/(.*)$", "^[./]"]
}
```
