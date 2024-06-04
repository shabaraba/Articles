# ChakraUIの

Created: 2022年4月6日 0:21
Updated: 2022年12月12日 0:21
Published: No
URL: https://blog.shaba.dev/posts/

```tsx
export const CommandLine: React.FC = () => {
	const inputElement = useRef<HTMLInputElement>(null)
  const { histories, setHistories } = useContext(CommandContext)
  const [ closed, setClosed ] = useState(false)
  const [ command, setCommand ] = useState("")

  inputElement.current?.focus()

  const onInput = (e:any) => {
    console.log(inputElement.current.value)
    if (e.key != 'Enter') return
    console.log("continue")

    setHistories([...histories, inputElement.current.value])
    setClosed(true)
    setCommand(inputElement.current.value)
  }

  return <Text contentEditable={true} autoCorrectk ref={inputElement} onKeyPress={onInput} />
}
```

これは

```tsx
export const CommandLine: React.FC = () => {
  const { histories, setHistories } = useContext(CommandContext)
  const [ closed, setClosed ] = useState(false)
  const [ command, setCommand ] = useState("")

  const onSubmit= (e:string) => {
    console.log(e)
    setHistories([...histories, e])
    setClosed(true)
    setCommand(e)
  }

  return (
    <Editable 
      defaultValue=""
      ref={inputElement}
      onSubmit={onSubmit}
      startWithEditView={true}
    >
      <EditablePreview />
      <EditableInput/>
    </Editable>
  )
```

こう書ける

useRef不要なのでスッキリ。