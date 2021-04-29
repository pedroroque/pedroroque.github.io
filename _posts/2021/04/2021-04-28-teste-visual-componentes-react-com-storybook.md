---
layout: post
title: Teste visual de componentes React com StoryBook
date: 2021-04-28 20:00:00 -0300
categories: react.js testing storyboard
---

Durante o desenvolvimento de uma aplicação web, muitas vezes recebemos os requisitos para construir um novo componente, sem que a página onde será utilizado esteja ainda definida.

O que muita gente acaba por fazer é criar uma página de teste, só para ter algo para mostrar para os stakeholders.

Uma alternativa melhor é a utilização do [StoryBook](https://storybook.js.org/). Trata-se de uma ferramenta que permite o teste em isolamento de componentes web, e está disponivel, entre outros, para React, Vue e Angular.

A ferramente deve ser instalada num projeto existente. Na raiz do projeto é só executar:

{% highlight ssh %}
npx sb init
{% endhighlight %}

No processo de instalação, é feita uma análise do projeto e sua configuração, para que sejam instaladas as depêndencias mais adequadas ao projeto.

Depois da instalação concluída, ficamos com um novo script para executar o nosso Storyboard:

{% highlight ssh %}
npm run storybook
{% endhighlight %}

Por default, o StoryBoard fica disponivel no endereço http://localhost:6006 
![Página inicial StoryBook](/assets/images/2021/04/28-sb-main-page.png)

Na estrutura do projeto é criada uma nova pasta, com alguma stories de exemplo:

![Pasta stories](/assets/images/2021/04/28-sb-stories-folder.png)

O componente que iremos testar é um formulário para entrada de email:

{% highlight typescript %}
import react, { useState } from "react";
import css from "./EmailForm.module.css";

export interface EmailFormProps {
  initialValue: string;
  message: string;
  onSubmit: (email: string) => void;
};

export const EmailForm = (props: EmailFormProps) => {
  const [email, setEmail] = useState(props.initialValue);
  const emailParts = countEmailParts(email);

  return (<section className={css.container}>
    <div className={css.spectrum} aria-hidden>
      {Array.from(Array(5)).map((_, i) => (<div className={i + 1 <= emailParts ? css.barActive : css.bar} key={i}></div>))}
    </div>
    <header className={css.header}>
      <h2>{props.message}</h2>
    </header>
    <input
      className={css.email}
      type="email"
      placeholder="your email"
      value={email}
      onChange={(evt) => setEmail(evt.target.value)}
    />
    <button className={css.submit}>Sign up</button>
  </section>)
}

const countEmailParts = (email: string): number => {
  if (/@.+\..{2,}$/.test(email)) {
    return 5
  } else if (/@.+\..?$/.test(email)) {
    return 4
  } else if (/@.+$/.test(email)) {
    return 3
  } else if (/@/.test(email)) {
    return 2
  } else if (/.+/.test(email)) {
    return 1
  } else {
    return 0
  }
}
{% endhighlight %}

Os testes do componente são criados num arquivo que siga o padrão *.stories.tsx (ou jsx se estivermos trabalhando direto com JS)

{% highlight javascript %}
import { Story, Meta } from '@storybook/react';
import { EmailForm, EmailFormProps } from "../components/Email/EmailForm";

export default {
  title: 'Components/EmailForm',
  component: EmailForm,
} as Meta;

const Template: Story<EmailFormProps> = (args) => <EmailForm {...args} />

export const Empty = Template.bind({});
Empty.args = { initialValue: "", message: "SUBSCRIBE!!", onSubmit: (e) => { } }

export const OnePart = Template.bind({});
OnePart.args = { initialValue: "pedro", message: "SUBSCRIBE!!", onSubmit: (e) => { } }

export const TwoParts = Template.bind({});
TwoParts.args = { initialValue: "pedro.roque@", message: "SUBSCRIBE!!", onSubmit: (e) => { } }

export const ThreeParts = Template.bind({});
ThreeParts.args = { initialValue: "pedro.roque@outlook", message: "SUBSCRIBE!!", onSubmit: (e) => { } }

export const FourParts = Template.bind({});
FourParts.args = { initialValue: "pedro.roque@outlook.", message: "SUBSCRIBE!!", onSubmit: (e) => { } }

export const FiveParts = Template.bind({});
FiveParts.args = { initialValue: "pedro.roque@outlook.com", message: "SUBSCRIBE!!", onSubmit: (e) => { } }
{% endhighlight %}

Na definição da story, começamos por criar os metadados, que serão usados para apresentar a story na página do StoryBook.

Depois, criamos o template do componente que vamos testar.

Finalmente, exportamos os cenários que queremos testar, definindo para cada um as props do componente.

Na página do StoryBook, podemos agora ver a renderização do componente:

![Visualização StoryBook](/assets/images/2021/04/28-sb-final.png)

Fica assim muito mais fácil e rápido criar algo que possa ser mostrado para os stakeholders e com isso tornar o ciclo de feedback mais rápido e eficaz.
