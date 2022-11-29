```toc
```


## Fix for Abstract Control to Form Control Issue

- Prevents issue on Casting here. 
- Alternative is $any(control) _this doesn't work with strict templating_

```typescript
@Pipe({  
  name: 'formControl'  
})  
export class FormControlPipe implements PipeTransform {  
  transform(value: AbstractControl): FormControl {  
    return value as FormControl;  
  }  
}
```

```html
<ras-currency-input [control]="term.get('amountCurrency') | formControl"></ras-currency-input>
```
