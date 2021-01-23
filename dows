<?php

namespace App\Http\Controllers\Member;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

use App\Account;
use App\Profile;
use App\Product;
use App\Transaction;

class MemberController extends Controller
{
    public function get_data(){
        // get detail account
        $account = Account::find(auth()->user()->id);
        // get account's profile picture (filename of picture = id)
        $img = Profile::where('user_id',auth()->user()->id)->first();

        $profile = [
            'name'=>$account->name,
            'place'=>$account->place,
            'wa'=>$account->wa,
            'ig'=>$account->ig,
            'ecommerce'=>$account->ecommerce,
            // nama file foto profile
            'img'=>$img->id."."."$img->type",
        ];

        return response()->json(compact('profile'));
    }

    public function edit_profile_picture(Request $request){
        
        // profile was created after the user get account (verified)
        $profile = Profile::where('user_id',auth()->user()->id)->first();

        if($profile->type==null && $request->file('file')==null){
            // 'type' will null for the first time, and required a file for the first profile update
            return response(['errors'=>'The profile picture is required'],422);
        }
        elseif($request->file('file')==null) {
            // pass, because the file is not required if user already had profile picture
            return response('',200);
        }

        $file = $request->file('file');

        // condition = false, if the uploaded file is not an image
        if (getimagesize($file)===false) {
            return response(['errors'=>"The file must be an image"],422);
        }
        elseif($file->getSize()>4100000){
            return response(['errors'=>"The maximum file size is 4 MB"],422);
        }

        // delete the old file
        if (file_exists("img/profile/$profile->id.$profile->type")) {
            unlink("img/profile/$profile->id.$profile->type");
        }
        // update the extension of file
        $profile->update([
            'type'=> $file->getClientOriginalExtension()
        ]);

        // move the new file to "profile" folder with uuid as file name
        $file->move('img/profile',$profile->id.".".$profile->type);

        return response()->json(['result'=>'success upload']);     
    }

    public function edit_profile(Request $request){
        request()->validate([
            'name'=> 'required|min:3|max:255',
            'place'=> 'required|min:3|max:255',
            'wa'=> 'required|min:10|regex:/^[0-9]+$/|max:15',
            'ig'=> 'required|min:3|max:255',
            'ecommerce'=> 'required|min:10|max:1000',
        ]);

       // get and update account detail
        $account = Account::where('id', auth()->user()->id)->first();
        $account->update([
            'name'=>$request->name,
            'place'=>$request->place,
            'wa'=>$request->wa,
            'ig'=>$request->ig,
            'ecommerce'=>$request->ecommerce
        ]);

        return response()->json(['result'=>'success update']);
    }

    public function create_product(Request $request){
        request()->validate([
            'name'=> 'required|min:3|max:255',
            'price'=> 'required|numeric|min:10000',
            'ig'=> 'required|min:10|max:1000',
            'ecommerce'=> 'required|min:10|max:1000',
            'description'=> 'required|min:3|max:1000',
            'file'=>'required|image|max:10000|'
        ]);

        $file= $request->file('file');

        $net_price = $request->price;
        $gross = $net_price*1.3;
        $discount = $net_price*0.23;
        $profit = ($gross-$discount-$net_price);

        $product = Product::create([
            'user_id'=>auth()->user()->id,
            'name'=> $request->name,
            'net_price'=>$net_price,
            'gross'=>$gross,
            'discount'=>$discount,
            'profit'=>$profit,
            'ig'=>$request->ig,
            'ecommerce'=>$request->ecommerce,
            'description'=> $request->description,
            'file_type'=>$file->getClientOriginalExtension()
        ]);
        
        // move the file to "product" folder with uuid as file name
        $file->move('img/product',$product->id.".".$product->file_type);

        return response()->json('sukses');
    }

    public function get_products(){
        $result = Product::where('user_id',auth()->user()->id)->get();;

        $product = [];
        foreach ($result as $value) {
            $product[] = [
                'id'=> $value->id,
                'name'=> $value->name,
                'price'=> 'Rp. '.number_format(ceil($value->net_price),0,",","."),
                'ig'=>$value->ig,
                'ecommerce'=>$value->ecommerce,
                'description'=> $value->description,
                'img' => $value->id.".".$value->file_type
            ];
        }

        return response()->json(compact('product'));
    }

    public function show_product(Product $id, Request $request){
        if ($id->user_id != auth()->user()->id) {
            return response('',401);
        }

        $product = [
            'name'=> $id->name,
            'price'=> ceil($id->net_price),
            'ig'=>$id->ig,
            'ecommerce'=>$id->ecommerce,
            'description'=> $id->description,
            'img' => $id->id.".".$id->file_type
        ];
        
        return response()->json(compact('product'));
    }

    public function edit_product(Product $id, Request $request){
        request()->validate([
            'name'=> 'required|min:3|max:255',
            'price'=> 'required|numeric|min:10000',
            'ig'=> 'required|min:10|max:1000',
            'ecommerce'=> 'required|min:10|max:1000',
            'description'=> 'required|min:3|max:1000',
        ]);

        // if want to change product's picture
        if ($request->file('file')!=null) {
            $file = $request->file('file');
    
            // condition = false, if the uploaded file is not an image
            if (getimagesize($file)===false) {
                return response(['errors'=>['file'=>"The file must be an image"]],422);
            }
            elseif($file->getSize()>10100000){
                return response(['errors'=>['file'=>"The maximum file size is 10 MB"]],422);
            }
        }

        if ($id->user_id != auth()->user()->id) {
            return response('',401);
        }

        // delete the old file if user upload the new file
        if(isset($file) && file_exists("img/product/$id->id.$id->file_type")){
            unlink("img/product/$id->id.$id->file_type");
        }

        $net_price = $request->price;
        $gross = $net_price*1.3;
        $discount = $net_price*0.23;
        $profit = ($gross-$discount-$net_price);

        // update the product
        $id->update([
            'name'=> $request->name,
            'net_price'=>$net_price,
            'gross'=>$gross,
            'discount'=>$discount,
            'profit'=>$profit,
            'ig'=>$request->ig,
            'ecommerce'=>$request->ecommerce,
            'description'=> $request->description,
            'file_type'=>isset($file)?$file->getClientOriginalExtension():$id->file_type
        ]);

        if (isset($file)) {
            // move the file to "product" folder with uuid as file name
            $file->move('img/product',$id->id.".".$id->file_type);
        }

        return response()->json(compact('id'));
    }

    public function delete_product(Product $id){
        if ($id->user_id != auth()->user()->id) {
            return response('',401);
        }
        // delete the product image file
        if(file_exists("img/product/$id->id.$id->file_type")){
            unlink("img/product/$id->id.$id->file_type");
        }
        // delete the product from product table
        $id->delete();
        return 'sukses';
    }

    // list umkm & product for landing page umkm
    public function all_umkm(){
        $all_umkm = Account::where('role','member')->get();

        // only umkm who has biodata & product will be show
        $umkm  = [];
        $id_umkm = 1;
        foreach ($all_umkm as $key => $value) {
            if (!empty($value->product[0]) && !empty($value->profile->type)) {

                $product = [];
                $id_product = 1;
                foreach ($value->product as $data) {
                    $product[] = [
                        'id'=> "$id_umkm.product.$id_product",
                        'img' => $data->id.'.'.$data->file_type,
                        'name' => $data->name,
                        'price' => 'Rp. '.number_format($data->gross,0,",","."),
                        'description'=> $data->description
                    ];
                    $id_product++;
                }

                $umkm[] = [
                    'id'=>$id_umkm,
                    'name'=>$value->name,
                    'place'=>$value->place,
                    'ig'=>$value->ig,
                    'ecommerce'=>$value->ecommerce,
                    'logo'=>$value->profile->id.'.'.$value->profile->type,
                    'products'=>$product
                ];
                $id_umkm++;
            }
        }
        return response()->json(compact('umkm'));
    }
    // list product for landing page product
    public function all_product(){
        $data_product = Product::get();

        $product = [];
        $id_product = 1;
        foreach ($data_product as $value) {
            $umkm = $value->umkm;
            $umkm = [
                'name'=>$umkm->name,
                'place'=>$umkm->place,
                'ig'=>$umkm->ig,
                'ecommerce'=>$umkm->ecommerce,
                'logo'=>$umkm->profile->id.'.'.$umkm->profile->type,
            ];

            $product[] = [
                'id'=> $id_product,
                'name'=>$value->name,
                'img'=>$value->id.'.'.$value->file_type,
                'price'=>'Rp. '.number_format($value->gross,0,",","."),
                'ig'=>$value->ig,
                'ecommerce'=>$value->ecommerce,
                'description'=>$value->description,
                'umkm'=>$umkm
            ];
            $id_product++;
        }
        return response()->json(compact('product'));
    }

    // create transaction for member
    public function transaction(Request $request){
        request()->validate([
            'product_name' => 'required|min:3',
            'net_price' => 'required|numeric|min:1000',
            'ongkir' => 'required|numeric|min:1000',
            'file' => 'required|image|max:2000|',
        ]);

        $net_price = $request->net_price;
        $gross = $net_price*1.3;
        $discount = $net_price*0.23;
        $profit = ($gross-$discount-$net_price);

        // store to database
        $transaction = Transaction::create([
            'user_id'=>auth()->user()->id,
            'product_name' => request()->product_name,
            'net_price'=>$net_price,
            'gross'=>$gross,
            'discount'=>$discount,
            'profit'=>$profit,
            'ongkir' => request()->ongkir,
        ]);

        $file = request()->file('file');

        // store name file. filename = id (uuid).extension (file_type)
        $transaction->update([
            'file_type' => $file->getClientOriginalExtension()
        ]);

        // move uploaded file
        $file->move('img/transaction',$transaction->id.'.'.$transaction->file_type);
        return response()->json('sukses');
    }
}
