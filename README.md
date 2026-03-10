# Garatuja

### 10/03/26

# class Pessoa {
    constructor(protected _name : string){ 
    }
    get name():string{
        return this.name;
    }

    // private name :string
    // constructor(name:string){
    //     this.name = name
    // }
}
let p1 = new Pessoa('joao')
