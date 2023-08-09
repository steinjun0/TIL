https://github.com/TanStack/query/discussions/1648
```tsx
    initialData: () => {
        const data = initialData()
        if (data) {
          return {
            pageParams: [undefined],
            pages: [data]
          }
        }
      },
```

위와 같이 initialData를 객체 형태로 반환해야한다. 그냥 데이터만(pages) 넣으면 타입이 안맞는다.