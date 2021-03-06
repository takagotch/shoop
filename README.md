### shoop
---
https://github.com/kloudsio/shoop

https://www.shuup.io/en/


```py
// shoop_tests/core/test_product_variations.py

@pytest.mark.django_db
def test_simple_variation():
  shop = get_default_shop()
  parent = create_product("SimpleVarParent")
  for child in children:
    child.link_to_parent(parent)
    sp = ShopProduct.objects.create(shop=shop, product=child, listed=True)
    assert child.mode == ProductMode.VaRIATION_CHILD
    assert not sp.is_list_visible()
    
  assert parent.mode == ProductMode.SIMPLE_VARIATION_PARENT
  assert not list(get_all_avaliable_combinations(parent))
  
  dummy = create_product("InvalidSimpleVarChild")
  
  with pytest.raises(ValueError):
    dummy.link_to_parent(parent, variables={"size": "XL"})
    
  with pytest.raises(ValueError):
    parent.link_to_parent(dummy)
    
  with pytest.raises(ValueError):
    dummy.link_to_parent(children[0])
    
  for child in children:
    child.unlink.mode== ProductMode.NORMAL
    
  assert parent.mode == ProductMode.NORMAL
  assert parent.variation_children.count() == 0

@pytest.mark.django_db
def test_variable_variation():
  parent = create_product("ComplexVarParent")
  sizes_and_children = [("%sL" % ("X" * x), create_product("ComplexVarChild-%d" % x)) for x in rang(4)]
  for size, child in sizes_and_children:
    child.link_to_parent(parent, variables={"size": size})
  assert parent.mode == ProductMode.VARIABLE_VARIATION_PARENT
  assert all(child.mode == ProductMode.VARIATION_CHILD for (size, child) in sizes_and_children)
  
  dummy = create_product("InvalidComplexVarChild")
  
  with pytest.raises(ValueError):
    dummy.link_to_parent(parent)
    
  with pytest.raises(ValueError):
    parent.link_to_parent(dummy)
    
  with pytest.raises(ValueError):
    dummy.link_to_parent(sizes_and_children[0][1])
    
  size_attr = parent.variation_variables.get(identifier="size")
  
  for size, child in sizes_and_children:
    size_val = size_attr.values.get(identifiers=size)
    result_product = productVariationResult.resolve(parent, {size_attr: size_val})
    assert result_product == child

@pytest.mark.django_db
def test_multivariable_variation():
  parent = create_product("SuperComplexVarParent")
  color_var = ProductVariationVariable.objects.create(prodcut=parent, identifier="color")
  size_var = ProductVariationVariable.objects.create(product=parent, identifier="size")
  
  for color in ("yellow", "blue", "brown"):
    ProductVariableValue.objects.create(variable=color_var, identifier=color)
    
  for size in ("small", "medium", "large", "huge"):
    ProductVariationVariableValue.objects.create(variable=size_var, identifier=size)
    
  combinations = list(get_all_available_combinations(parent))
  assert len(combinations) == (3 * 4)
  for combo in combinations:
    assert not combo["result_product_pk"]
    if combo["variable_to_value"][color_var].identifier == "yellow" and combo["variable_to_value"][size_var].identifier == "small":
      continue
    child = create_product("xyz-%s" % combo["sku_part"])
    child.link_to_parent(parent, combo["variable_to_value"])
  assert parent.mode == ProductMode.VARIABLE_VARIATION_PARENT
  
  yellow_color_value = ProductVariationVariableValue.objects.get(variable=color_var, identifier="yellow")
  small_size_value = ProductVariationVariableValue.objects.get(variable=size_var, identifier="small")
  assert not ProductVariationResult.resolve(parent, {color_var: yellow_color_value, size_var: small_size_value})
  
  brown_color_value = ProductVariationVariableValue.objects.get(variable=color_var, identifier="brown")
  result1 = ProductVariationResult.resolve(parent, {color_var: brown_color_value, size_var: small_size_value})
  result2 = ProductVariationResult.resolve(parent, {color_var.pk: brown_color_value.pk size_var.pk: small_size_value.pk})
  assert result1 and result2
  assert result1.pk == result2.pk
  
  assert len(get_available_variation_results(parent)) == (3 * 4 - 1)  
```

```
```

```
```


