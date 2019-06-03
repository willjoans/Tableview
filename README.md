# Tableview


import UIKit
import RealmSwift
class MenuAllItemCell: UITableViewCell {
    @IBOutlet var lblPrice: UILabel!
    @IBOutlet var lblIteamsName: UILabel!
    @IBOutlet var btnMines: UIButton!
    @IBOutlet var btnPlues: UIButton!
    @IBOutlet var lblCount: UILabel!
}
class HederViewCell: UITableViewCell {
    @IBOutlet var lblTitle: UILabel!
}


class MenuAllVC: UIViewController,UITableViewDelegate,UITableViewDataSource {
    
    @IBOutlet var lblTitle: UILabel!
    @IBOutlet var lblAmount: UILabel!
    @IBOutlet var btnCart: UIButton!
    var ArrayCart : Results<VendorBean>?
    var realm = try! Realm()
    var ArraySectionHeader = [VendorAllBean]()
    var ArrayDataMemu = [VendorAllBean]()
    var BeanIteam = VendorBean()
    @IBOutlet var tblviewAllMeItem: UITableView!
    var ArrayMenu = [VendorBean]()
    var Vender_ID = ""
    var adultaValue = 0
    var ConutData = 0
    var PRice   = 0
    override func viewDidLoad() {
        super.viewDidLoad()
        if BeanIteam.firm_name != "<null>"{
            self.lblTitle.text = BeanIteam.firm_name
        }
        tblviewAllMeItem.delegate = self
        tblviewAllMeItem.dataSource = self
        self.navigationController?.isNavigationBarHidden = true
        self.GET_ALL_ITEAM()
         NotificationCenter.default.addObserver(self, selector: #selector(self.MenuAllData), name:.NOTIFICATION_ALLMENU, object: nil)
    }
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
         ArrayCart = realm.objects(VendorBean.self)
        for DicData in ArrayCart!{
            self.Vender_ID = DicData.fk_venders_id
        }
        if ArrayCart?.count != 0 {
            self.btnCart.setImage(UIImage(named:"ic_cart_3"), for: .normal)
        }else{
            self.btnCart.setImage(UIImage(named:"shopping-store-cart-"), for: .normal)
        }
    }
    @objc func MenuAllData() {
        self.PRice = 0
        self.GET_ALL_ITEAM()
    }
    @IBAction func TapTOBack(_ sender: UIButton) {
        self.dismiss(animated: true, completion: nil)
    }
    @IBAction func TapToCart(_ sender: Any) {
        let objStory : UIStoryboard = UIStoryboard(name: "Main", bundle: nil)
        let Cartvc = objStory.instantiateViewController(withIdentifier: "CartVC") as? CartVC
        Cartvc?.IsSelected = true
        self.present(Cartvc!, animated: true, completion: nil)
    }
    @IBAction func TapToPlues(_ sender: UIButton) {
        let cell = sender.superview?.superview?.superview as! MenuAllItemCell
        let indexPath = tblviewAllMeItem.indexPath(for: cell)
        
        let Bean = ArraySectionHeader[indexPath!.section].DataArrayVendor[indexPath!.row]
        
        if Vender_ID == Bean.fk_venders_id ||  Vender_ID == ""{
            if Bean.CountIteam  <= 9{
                Bean.CountIteam =  Bean.CountIteam + 1
                
                self.PRice = TO_INT(Bean.price) + self.PRice
                
                self.lblAmount.text = Currency + " " + TO_STRING(PRice) + ".0"
            }
//            let indexPath = IndexPath(item: sender.tag, section: 0)
//            tblviewAllMeItem.reloadRows(at: [indexPath], with: .none)
            self.tblviewAllMeItem.reloadData()
        }else{
            self.RemoveDataCart()
        }
    }
    @IBAction func TapToMines(_ sender: UIButton) {
        
        let cell = sender.superview?.superview?.superview as! MenuAllItemCell
        let indexPath = tblviewAllMeItem.indexPath(for: cell)
        
        let Bean = ArraySectionHeader[indexPath!.section].DataArrayVendor[indexPath!.row]
        
        if Vender_ID == Bean.fk_venders_id ||  Vender_ID == ""{
            if Bean.CountIteam  > 0{
                Bean.CountIteam =  Bean.CountIteam - 1
                self.PRice =  self.PRice - TO_INT(Bean.price)
                self.lblAmount.text = Currency + " " + TO_STRING(PRice) + ".0"
            }
//            let indexPath = IndexPath(item: sender.tag, section: 0)
//            tblviewAllMeItem.reloadRows(at: [indexPath], with: .none)
            self.tblviewAllMeItem.reloadData()

        }else{
            self.RemoveDataCart()
        }
        
    }
    @IBAction func TapTOAddToCart(_ sender: Any) {
        self.btnCart.setImage(UIImage(named:"ic_cart_3"), for: .normal)
        self.Data_Insert_Add_Cart()
        
        
    }
    
    func Data_Insert_Add_Cart(){

        do {
            for DataDicMenu  in ArraySectionHeader{
                
                let DicDataArray = DataDicMenu.DataArrayVendor
                 for DataDicInsert in DicDataArray{
                var StrDataiteam = 0
                
                for Dicupdate in ArrayCart!{
                    if Dicupdate.items_id == DataDicInsert.items_id{
                        StrDataiteam = Dicupdate.categories_Main_id
                    }
                }
                
                if StrDataiteam != 0{
                    print(DataDicInsert.CountIteam)
                    if DataDicInsert.CountIteam != 0{
                        let Bean = VendorBean()
                        Bean.categories_Main_id = StrDataiteam
                        Bean.items_id = DataDicInsert.items_id
                        Bean.fk_categories_id = DataDicInsert.fk_categories_id
                        Bean.fk_subcategory_id = DataDicInsert.fk_subcategory_id
                        Bean.fk_venders_id = DataDicInsert.fk_venders_id
                        Bean.name = DataDicInsert.name
                        Bean.price = DataDicInsert.price
                        Bean.image = DataDicInsert.image
                        Bean.Description = DataDicInsert.Description
                        Bean.CountIteam = DataDicInsert.CountIteam
                        try! realm.write {
                            realm.add(Bean, update: true)
                            print("UpdateData--->\(Bean)")
                        }
                    }
                    else{
                        try! realm.write {
                            let Bean = VendorBean()
                            Bean.categories_Main_id = StrDataiteam
                            Bean.items_id = "0"
                            realm.add(Bean, update: true)
                        }
                    }
                }else{
                    if DataDicInsert.CountIteam != 0{
                        let Bean = VendorBean()
                        Bean.categories_Main_id = DataDicInsert.incrementID()
                        Bean.items_id = DataDicInsert.items_id
                        Bean.fk_categories_id = DataDicInsert.fk_categories_id
                        Bean.fk_subcategory_id = DataDicInsert.fk_subcategory_id
                        Bean.fk_venders_id = DataDicInsert.fk_venders_id
                        Bean.name = DataDicInsert.name
                        Bean.price = DataDicInsert.price
                        Bean.image = DataDicInsert.image
                        Bean.Description = DataDicInsert.Description
                        Bean.CountIteam = DataDicInsert.CountIteam
                        try! realm.write {
                            realm.add(Bean)
                            print("InsertData--->\(Bean)")
                        }
                    }
                }
                JonAlert.show(message:"Sucessfully added Cart", andIcon: UIImage(named:"success"))
            }
        }
    }
        catch let error {
            print(".........Error in add User function.........")
            print(error.localizedDescription)
        }
    }

    func DataGetIteamCount(){
        let realm = try! Realm()
        try! realm.write {
            for  Dic in ArraySectionHeader {
                let DicDataArray = Dic.DataArrayVendor
                for MainDic in DicDataArray{
                
                    for CatrDic in ArrayCart!{
                    if CatrDic.items_id == MainDic.items_id{
                        MainDic.CountIteam = CatrDic.CountIteam
                        MainDic.price = CatrDic.price
                        self.PRice += MainDic.price*MainDic.CountIteam
                    }
                }
              }
            }
            self.lblAmount.text = Currency + " " + TO_STRING(PRice) + ".0"
            self.tblviewAllMeItem.reloadData()
        }
    }
    
    
    func RemoveDataCart(){
        let refreshAlert = UIAlertController(title: "Cart Conflict!", message: "Your cart consists items from another vendors,do you want to clear your old vendors items to add this new vendor items", preferredStyle: UIAlertControllerStyle.alert)
        
        refreshAlert.addAction(UIAlertAction(title: "YES", style: .destructive, handler: { (action: UIAlertAction!) in
            DispatchQueue.main.async {
                self.Vender_ID = ""
               self.btnCart.setImage(UIImage(named:"shopping-store-cart-"), for: .normal)
                let realms = try! Realm()
                try! realms.write {
                    let TempArry = realms.objects(VendorBean.self)
                    for MyBean in TempArry{
                        realms.delete(MyBean)
                    }
                }
            }
        }))
        
        refreshAlert.addAction(UIAlertAction(title: "NO", style: .default, handler: { (action: UIAlertAction!) in
            refreshAlert .dismiss(animated: true, completion: nil)
        }))
        
        present(refreshAlert, animated: true, completion: nil)
    }
    func numberOfSections(in tableView: UITableView) -> Int {
        return ArraySectionHeader.count
    }
    func tableView(_ tableView: UITableView, heightForHeaderInSection section: Int) -> CGFloat {
        return 45
        
    }
    func tableView(_ tableView: UITableView, viewForHeaderInSection section: Int) -> UIView?
    {
        let cellIncome = tableView.dequeueReusableCell(withIdentifier: "HederViewCell")as! HederViewCell
        let Bean = ArraySectionHeader[section]
       
        cellIncome.lblTitle.text = Bean.name
        return cellIncome
    }
    func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
        return 45
    }
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return ArraySectionHeader[section].DataArrayVendor.count
    }
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let Cell = tableView.dequeueReusableCell(withIdentifier: "MenuAllItemCell")as! MenuAllItemCell
        let Bean = ArraySectionHeader[indexPath.section].DataArrayVendor[indexPath.row]
        
        Cell.lblCount.text = String(Bean.CountIteam)
        Cell.lblIteamsName.text = Bean.name
        Cell.lblPrice.text =  Currency + " " + TO_STRING(Bean.price)
        Cell.btnMines.tag = indexPath.row
        Cell.btnPlues.tag = indexPath.row

        return Cell
    }
    
    
    
    
    
    
    func GET_ALL_ITEAM() {
        let CateGory_ID = TO_STRING(Pref.getObjectForKey(KCateGories_ID))
        let venders_id = TO_STRING(Pref.getObjectForKey(Kvenders_id))
        
        let params = ["authcode":"\(Status_Auth_Code)",
            "fk_vendors_id":"\(venders_id)",
            "fk_categories_id":"\(CateGory_ID)"]
        
        print(params)
        PSWebServiceAPI.ITEAMOBBYCATEGORY(params , completion: { (response) in
            
            let ResponceData = response as [String:AnyObject]
            print(ResponceData)
            let States = TO_INT(ResponceData["Status"])
            let MessageAlert = TO_STRING(ResponceData["Msg"])
            print(MessageAlert)
            
            if States == 1{
                 let ArrayDataVedor = ResponceData["Data"]as! [AnyObject]
                self.ArraySectionHeader.removeAll()
                for DicData in ArrayDataVedor{
                let Bean = VendorAllBean()
                Bean.name = TO_STRING(DicData["name"]as AnyObject)
                self.ArraySectionHeader.append(Bean)
                
                let DataArray = DicData["items"]as! [AnyObject]
                    
                    for DataGet in DataArray{
                        let BeanVendor = VendorBean()
                        BeanVendor.items_id = TO_STRING(DataGet["items_id"]as AnyObject)
                        BeanVendor.fk_categories_id = TO_STRING(DataGet["fk_categories_id"]as AnyObject)
                        BeanVendor.fk_subcategory_id = TO_STRING(DataGet["subcategory_id"]as AnyObject)
                       BeanVendor.fk_venders_id = TO_STRING(DataGet["fk_venders_id"]as AnyObject)
                       BeanVendor.name = TO_STRING(DataGet["name"]as AnyObject)
                       BeanVendor.price = TO_INT(DataGet["price"]as AnyObject)
                       BeanVendor.image = TO_STRING(DataGet["image"]as AnyObject)
                       BeanVendor.Description = TO_STRING(DataGet["description"]as AnyObject)
                       BeanVendor.CountIteam = 0
                       Bean.DataArrayVendor.append(BeanVendor)
                    }
                }
              self.tblviewAllMeItem.reloadData()
                if self.ArrayCart?.count != 0{
                    self.btnCart.setImage(UIImage(named:"ic_cart_3"), for: .normal)
                    DispatchQueue.main.async {
                        self.DataGetIteamCount()
                    }
                }else{
                    self.lblAmount.text = Currency + " " + "0.0"
                    self.btnCart.setImage(UIImage(named:"shopping-store-cart-"), for: .normal)
                }
            }
            
            else{
//                appDelegate.window?.rootViewController?.view.makeToast(message: "\(String(describing: "NO Data"))")
            }
        })
    }

}
