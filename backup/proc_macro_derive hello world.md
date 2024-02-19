- [builder](#builder)
  - [cargo.toml](#cargotoml)
  - [struct](#struct)
  - [lib.rs](#librs)
  - [验证结果](#验证结果)

# builder 

> 给定 `struct` 生成`builder`相关的结构

## cargo.toml

```toml
[package]
name = "builder"
version = "0.1.0"
edition = "2021"

[lib]
proc-macro = true

[dependencies]
proc-macro2 = "1.0"
quote = "1.0"
syn = { version = "2.0", features = ["full"] }
```

## struct

```rust
#[derive(builder::Builder)]
pub struct Ad {
	id: u64,
	cids: Vec<u64>,
	title: String,
	bid: f32,
}
```

## lib.rs

- `quote!`: 将`Rust`代码转换为`TokenStream`
- `parse_macro_input!` 输入的 `TokenStream` 解析成 `Rust` 代码的数据结构


```rust
#[proc_macro_derive(Builder)]
pub fn derive(input: proc_macro::TokenStream) -> proc_macro::TokenStream {
    let ast = syn::parse_macro_input!(input as syn::DeriveInput);

    let ident = ast.ident;

    let builder_ident = quote::format_ident!("{ident}Builder");

    // 获取所有的field 
    let fields = match ast.data.clone() {
        syn::Data::Struct(data) => data.fields,
        _ => panic!("only support struct"),
    };

    // 定义builder 的字段
    // id: std::option::Option<u64>
    let builder_fields = fields.iter().map(|field| {
        let field = field.clone();
        let id = field.ident.unwrap();
        let ty = field.ty;

        quote::quote! {
            #id: std::option::Option<#ty>
        }
    });

    // 初始化为None 
    // id: std::option::Option::None 
    let builder_defaults = fields.iter().map(|field| {
        let field = field.clone();
        let id = field.ident.unwrap();

        quote::quote! { #id: std::option::Option::None }
    });

    // 定义setter 函数
    //     pub fn id(&mut self, value: u64) -> &mut Self {
    //     self.id = std::option::Option::Some(value);
    //     self
    // }
    let setters = fields.iter().map(|field| {
        let field = field.clone();
        let ty = field.ty;
        let id = field.ident.unwrap();

        quote::quote! {
            pub fn #id(&mut self,value:#ty) -> &mut Self {
                self.#id = std::option::Option::Some(value);
                self
            }
        }
    });

    // 字段赋值
    //  id: self.id.clone().unwrap()
    let build_fields = fields.iter().map(|field| {
        let field = field.clone();
        let id = field.ident.unwrap();

        quote::quote! {
            #id: self.#id.clone().unwrap()
        }
    });

    let output = quote::quote! {
        // 定义builder struct 
        pub struct #builder_ident {
            #(#builder_fields),*
        }

        impl #builder_ident {
            #(#setters)*
        }

        impl #builder_ident {
            pub fn build(&mut self) -> std::result::Result<#ident, std::boxed::Box<dyn std::error::Error>> {
                std::result::Result::Ok(#ident {
                    #(#build_fields),*
                })
            }

        }

        impl #ident {
            pub fn builder() -> #builder_ident {
                #builder_ident {
                    #(#builder_defaults),*
                }
            }
        }
    };

    proc_macro::TokenStream::from(output)
}
```

## 验证结果 

```rust

#[derive(builder::Builder)]
pub struct Ad {
    id: u64,
    cids: Vec<u64>,
    title: String,
    bid: f32,
}

fn main() {
    let mut builder = Ad::builder();

    let ad = builder
        .bid(1.23)
        .id(193987829387)
        .cids(vec![1, 2, 3])
        .title("推广".to_string())
        .build();
}
```

> cargo expand 

```rust
#![feature(prelude_import)]
#[prelude_import]
use std::prelude::rust_2021::*;
#[macro_use]
extern crate std;
pub struct Ad {
    id: u64,
    cids: Vec<u64>,
    title: String,
    bid: f32,
}
pub struct AdBuilder {
    id: std::option::Option<u64>,
    cids: std::option::Option<Vec<u64>>,
    title: std::option::Option<String>,
    bid: std::option::Option<f32>,
}
impl AdBuilder {
    pub fn id(&mut self, value: u64) -> &mut Self {
        self.id = std::option::Option::Some(value);
        self
    }
    pub fn cids(&mut self, value: Vec<u64>) -> &mut Self {
        self.cids = std::option::Option::Some(value);
        self
    }
    pub fn title(&mut self, value: String) -> &mut Self {
        self.title = std::option::Option::Some(value);
        self
    }
    pub fn bid(&mut self, value: f32) -> &mut Self {
        self.bid = std::option::Option::Some(value);
        self
    }
}
impl AdBuilder {
    pub fn build(
        &mut self,
    ) -> std::result::Result<Ad, std::boxed::Box<dyn std::error::Error>> {
        std::result::Result::Ok(Ad {
            id: self.id.clone().unwrap(),
            cids: self.cids.clone().unwrap(),
            title: self.title.clone().unwrap(),
            bid: self.bid.clone().unwrap(),
        })
    }
}
impl Ad {
    pub fn builder() -> AdBuilder {
        AdBuilder {
            id: std::option::Option::None,
            cids: std::option::Option::None,
            title: std::option::Option::None,
            bid: std::option::Option::None,
        }
    }
}
fn main() {
    let mut builder = Ad::builder();
    let ad = builder
        .bid(1.23)
        .id(193987829387)
        .cids(<[_]>::into_vec(#[rustc_box] ::alloc::boxed::Box::new([1, 2, 3])))
        .title("推广".to_string())
        .build();
}
```