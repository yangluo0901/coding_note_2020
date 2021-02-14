# HTML/CSS/js/jquery

# 1. TroubleShooting

# 2. Note    

1. **Disable and gray out a link**    

```scss
.disable{
    color: currentColor;
    cursor: not-allowed;
    opacity: 0.5;
    text-decoration: none;
}
```

2. **Place an element center when its position is absolute**    

```css
.pager{
    position: absolute;
    bottom: 100px;
    left:0;
    right:0;
    margin: auto;
}
```

3. **Table row as link**:

   ```html
   <tr data-href = "<link>"> content </tr>
   ```

   in js file:

   ```javascript
   $("tr[data-href]").click ( function (){
   	document.location = $(this).data('href');
   	return false;
     })
   ```

4. **How to add closable block**

   ```html
   	 <div class="alert alert-success">
        <a class="close" href="#" data-dismiss="alert">◊</a>
        {{ message }}
      </div>
   
   ```

5. **get row number of cusor**:

   cursor.rowcount

6. **button as link**

   ```html
   <button class="btn btn-default" onclick="location.href='{% url "add_part_order" po_num=new_po_num %}'">
   ```

   

   

   