# Development Guide

Before starting, ensure your machine is having:

- Node >= 18
- pnpm >= 8

## Setup

Fork [Vexip UI](https://github.com/vexip-ui/vexip-ui) and clone to your local machine and install dependencies:

```sh
pnpm install # pnpm i
```

Then you need to build once the packages under `common` (**IMPORTANT**):

```sh
pnpm run build:common
```

## Component Development

We use a Vite project in `dev-server` a development server.

You can use the following command to start development server for specify component:

```sh
pnpm run dev [component]
```

After the server is successfully started, the demos of the components specified under `docs/demos` will be used as development cases.

The development server uses `8008` port and Chinese demos by default, you can add `-p` and `-l` parameters to the command to specify the port and language respectively:

```sh
pnpm run dev [component] -p [port] -l [language]
```

## Documentation Development

We use [VitePress](https://vitepress.dev/) as the documentation framework. You can start it locally with the following command:

```sh
pnpm run dev:docs
```

## Create New Component

You can quickly create a new component using template files:

```sh
pnpm run create [component]
```

Wait patiently for the files to be created, then you can check the files in the following locations:

- `components/[component]/index.ts`
- `components/[component]/props.ts`
- `components/[component]/css.ts`
- `components/[component]/style.ts`
- `components/[component]/[component].vue`
- `components/[component]/tests/ssr.spec.tsx`
- `components/[component]/tests/[component].spec.tsx`
- `docs/demos/[component]/basis/demo.en-US.vue`
- `docs/demos/[component]/basis/demo.zh-CN.vue`
- `docs/en-US/component/[component].md`
- `docs/zh-CN/component/[component].md`
- `style/[component].scss`

In addition to the above template files, we also have some file name conventions:

- `components/[component]/symbol.ts` is used to define types and some common constants and variables
- `components/[component]/helpers.ts` is used to define some dedicated helper methods
- `components/[component]/hooks.ts` is used to define some dedicated hook methods

After confirming, you can execute the bootstrap command:

```sh
pnpm run bootstrap
```

After finished, you can start developing the component and its documentation.
