# Book Custom Class
berikut ini merupakan catatan skrip PHP yang digunakan untuk membuat plugin book custom class di SLiMS. Plugin ini digunakan untuk membuat klasifikasi kustom terkait buku seperti buku berdasarkan prodi, kelas, jurusan dll. Adapaun praktek nya bisa ditonton di kanal Youtube saya

## Struktur direktori plugin
```bash
- book_custom_class/
-- book_custom_class.plugin.php
-- pages/
--- custom_class.php
-- migration/
--- 1_CreateTable.php
```

### File biblio_prodi.plugin.php
##### Meramu isi file 
```php
<?php
/**
 * Plugin Name: Book Custom Class
 * Plugin URI: -
 * Description: -
 * Version: 1.0.0
 * Author: Drajat Hasan
 * Author URI: https://t.me/drajathasan
 */

use SLiMS\{DB,Plugins,Json};

// get plugin instance
$plugin = Plugins::getInstance();

```
##### Mendaftarkan menu pada module Master File
```php
// Menu for master data
$plugin->registerMenu('master_file', 'Pengklompokan CUstom', __DIR__ . '/pages/custom_class.php');
```
##### Membuat folder pages/ dan meletakan file custom_class.php
buatlah folder beranama pages/ pada folder plugin book_custom_class, setelah itu salin skrip dibawah ini dan berinama custom_class.php
```php
<?php
/**
 * Copyright (C) 2007,2008  Arie Nugraha (dicarve@yahoo.com)
 *
 * This program is free software; you can redistribute it and/or modify
 * it under the terms of the GNU General Public License as published by
 * the Free Software Foundation; either version 3 of the License, or
 * (at your option) any later version.
 *
 * This program is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 * GNU General Public License for more details.
 *
 * You should have received a copy of the GNU General Public License
 * along with this program; if not, write to the Free Software
 * Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA  02110-1301  USA
 *
 */

/* GMD Management section */

// key to authenticate
defined('INDEX_AUTH') or die('Direct access is not allowed');

require SB.'admin/default/session.inc.php';
require SB.'admin/default/session_check.inc.php';
require SIMBIO.'simbio_GUI/table/simbio_table.inc.php';
require SIMBIO.'simbio_GUI/form_maker/simbio_form_table_AJAX.inc.php';
require SIMBIO.'simbio_GUI/paging/simbio_paging.inc.php';
require SIMBIO.'simbio_DB/datagrid/simbio_dbgrid.inc.php';
require SIMBIO.'simbio_DB/simbio_dbop.inc.php';

// privileges checking
$can_read = utility::havePrivilege('master_file', 'r');
$can_write = utility::havePrivilege('master_file', 'w');

if (!$can_read) {
    die('<div class="errorBox">'.__('You don\'t have enough privileges to view this section').'</div>');
}

function httpQuery($query = [])
{
    return $_SERVER['PHP_SELF'] . '?' . http_build_query(array_unique(array_merge($_GET, $query)));
}

/* RECORD OPERATION */
if (isset($_POST['saveData']) AND $can_read AND $can_write) {
    $name = trim(strip_tags($_POST['name']));
    // check form validity
    if (empty($name)) {
        utility::jsToastr( 'Error', 'Nama tidak boleh kosong!', 'warning');
        exit();
    } else {
        $data['name'] = $dbs->escape_string($name);
        $data['created_at'] = date('Y-m-d');

        // create sql op object
        $sql_op = new simbio_dbop($dbs);
        if (isset($_POST['updateRecordID'])) {
            /* UPDATE RECORD MODE */
            unset($data['created_at']);
            // filter update record ID
            $updateRecordID = $dbs->escape_string(trim($_POST['updateRecordID']));
            // update the data
            $update = $sql_op->update('mst_custom_class', $data, 'id='.$updateRecordID);
            if ($update) {
                utility::jsToastr('Sukses', 'Berhasil memperbaharui data', 'success');
                echo '<script type="text/javascript">parent.jQuery(\'#mainContent\').simbioAJAX(\''.httpQuery(['action' => 'list']).'\');</script>';
            } else { toastr('Gagal memperbaharui data'."\nDEBUG : ".$sql_op->error)->error(); }
            exit();
        } else {
            /* INSERT RECORD MODE */
            // insert the data
            if ($sql_op->insert('mst_custom_class', $data)) {
                utility::jsToastr('Sukses', 'Berhasil menambah data', 'success');
                echo '<script type="text/javascript">parent.jQuery(\'#mainContent\').simbioAJAX(\''.httpQuery(['action' => 'list']).'\');</script>';
            } else { 
                utility::jsToastr( 'Gaga;', 'Gagal memperbaharui data'."\nDEBUG : ".$sql_op->error, 'error');
            }
            exit();
        }
    }
    exit();
} else if (isset($_POST['itemID']) AND !empty($_POST['itemID']) AND isset($_POST['itemAction'])) {
    if (!($can_read AND $can_write)) {
        die();
    }
    /* DATA DELETION PROCESS */
    $sql_op = new simbio_dbop($dbs);
    $failed_array = array();
    $error_num = 0;
    $still_have_biblio = array();
    if (!is_array($_POST['itemID'])) {
        // make an array
        $_POST['itemID'] = array((integer)$_POST['itemID']);
    }
    // loop array
    foreach ($_POST['itemID'] as $itemID) {
        $itemID = (integer)$itemID;
        if (!$sql_op->delete('mst_custom_class', 'id='.$itemID)) {
            $error_num++;
        }
    }

    // error alerting
    if ($error_num == 0) {
        toastr('Sukses menghapus data')->success();
        echo '<script type="text/javascript">parent.jQuery(\'#mainContent\').simbioAJAX(\''.$_SERVER['PHP_SELF'].'?'.$_POST['lastQueryStr'].'\');</script>';
    } else {
        toastr('Gagal menghapus data')->error();
        echo '<script type="text/javascript">parent.jQuery(\'#mainContent\').simbioAJAX(\''.$_SERVER['PHP_SELF'].'?'.$_POST['lastQueryStr'].'\');</script>';
    }
    exit();
}
/* RECORD OPERATION END */

/* search form */
?>
<div class="menuBox">
<div class="menuBoxInner masterFileIcon">
	<div class="per_title">
	    <h2>Pengelompokan Kustom</h2>
  </div>
	<div class="sub_section">
    <div class="btn-group">
      <a href="<?= httpQuery() ?>" class="btn btn-default">Daftar Kelompok</a>
      <a href="<?= httpQuery(['action' => 'detail']) ?>" class="btn btn-default">Tambah Kelompok Baru</a>
    </div>
    <form name="search" action="<?= httpQuery() ?>" id="search" method="get" class="form-inline"><?php echo __('Search'); ?> 
      <input type="text" name="keywords" class="form-control col-3" />
      <input type="submit" id="doSearch" value="<?php echo __('Search'); ?>" class="s-btn btn btn-default" />
    </form>
  </div>
</div>
</div>
<?php
/* search form end */
/* main content */
if (isset($_POST['detail']) OR (isset($_GET['action']) AND $_GET['action'] == 'detail')) {
    if (!($can_read AND $can_write)) {
        die('<div class="errorBox">'.__('You don\'t have enough privileges to view this section').'</div>');
    }
    /* RECORD FORM */
    $itemID = (integer)isset($_POST['itemID'])?$_POST['itemID']:0;
    $rec_q = $dbs->query('SELECT * FROM mst_custom_class WHERE id='.$itemID);
    $rec_d = $rec_q->fetch_assoc();

    // create new instance
    $form = new simbio_form_table_AJAX('mainForm', $_SERVER['PHP_SELF'].'?'.$_SERVER['QUERY_STRING'], 'post');
    $form->submit_button_attr = 'name="saveData" value="'.__('Save').'" class="s-btn btn btn-default"';

    // form table attributes
    $form->table_attr = 'id="dataList" class="s-table table"';
    $form->table_header_attr = 'class="alterCell font-weight-bold"';
    $form->table_content_attr = 'class="alterCell2"';

    // edit mode flag set
    if ($rec_q->num_rows > 0) {
        $form->edit_mode = true;
        // record ID for delete process
        $form->record_id = $itemID;
        // form record title
        $form->record_title = $rec_d['name'];
        // submit button attribute
        $form->submit_button_attr = 'name="saveData" value="'.__('Update').'" class="s-btn btn btn-primary"';
    }

    /* Form Element(s) */
    // group name
    $form->addTextField('text', 'name', 'Nama kelompok*', $rec_d['name']??'', 'style="width: 60%;" class="form-control"');

    // edit mode messagge
    if ($form->edit_mode) {
        echo '<div class="infoBox">Anda akan mengganti nama kelompok : <b>'.$rec_d['name'].'</b>  <br />'.__('Last Update').' '.$rec_d['updated_at'].'</div>'; //mfc
    }
    // print out the form object
    echo $form->printOut();
} else {
    /* GMD LIST */
    // table spec
    $table_spec = 'mst_custom_class';

    // create datagrid
    $datagrid = new simbio_datagrid();
    if ($can_read AND $can_write) {
        $datagrid->setSQLColumn('id',
            'name AS \'Nama Kelompok\'',
            'updated_at AS \''.__('Last Update').'\'');
    } else {
        $datagrid->setSQLColumn('name AS \'Nama Kelompok\'',
            'name AS \'Nama Kelompok\'',
            'updated_at AS \''.__('Last Update').'\'');
    }
    $datagrid->setSQLorder('updated_at DESC');

    // is there any search
    if (isset($_GET['keywords']) AND $_GET['keywords']) {
       $keywords = utility::filterData('keywords', 'get', true, true, true);
       $datagrid->setSQLCriteria("name LIKE '%$keywords%'");
    }

    // set table and table header attributes
    $datagrid->table_attr = 'id="dataList" class="s-table table"';
    $datagrid->table_header_attr = 'class="dataListHeader" style="font-weight: bold;"';
    // set delete proccess URL
    $datagrid->chbox_form_URL = httpQuery();

    // put the result into variables
    $datagrid_result = $datagrid->createDataGrid($dbs, $table_spec, 20, ($can_read AND $can_write));
    if (isset($_GET['keywords']) AND $_GET['keywords']) {
        $msg = str_replace('{result->num_rows}', $datagrid->num_rows, __('Found <strong>{result->num_rows}</strong> from your keywords')); //mfc
        echo '<div class="infoBox">'.$msg.' : "'.htmlspecialchars($_GET['keywords']).'"</div>';
    }

    echo $datagrid_result;
}
/* main content end */
```
##### Membuat hook untuk membuat kustom form dan menangkap isi dari kustom form yang sudah dibuat
Menangkap data kustom form dan mendaftarkan dalam variabel $custom_data
```php
// registering menus or hook
$plugin->register('advance_custom_field_data', function(&$custom_data){
    if (isset($_POST['custom_class']) && count($_POST['custom_class'])) $custom_data['custom_class'] = (string)Json::stringify($_POST['custom_class']);
    if (isset($_POST['custom_class']) && !count($_POST['custom_class'])) $custom_data['custom_class'] = '[]';
});
```
Membuat kustom form
```php
$plugin->register('advance_custom_field_form', function($form, &$js){

    // List
    $customClass = DB::getInstance()->query('SELECT `id`,`name` FROM `mst_custom_class`');

    $options = [];
    while ($content = $customClass->fetch(PDO::FETCH_NUM)) {
        $options[] = $content;
    }

    $list = '<option value="0">Pilih</option>';
    foreach ($options as $option) {
        list($value,$label) = $option;
        $list .= '<option value="' . $value . '">' . $label . '</option>' . "\n";
    }

    $buttonList = '';
    if (isset($_REQUEST['itemID']))
    {
        // List
        $biblioCustomClass = DB::getInstance()->prepare('SELECT `custom_class` FROM `biblio_custom` WHERE `biblio_id` = ?');
        $biblioCustomClass->execute([$_REQUEST['itemID']]);
        $biblioCustomClassData = [];
        if ($biblioCustomClass->rowCount() > 0) $biblioCustomClassData = Json::parse($biblioCustomClass->fetch(PDO::FETCH_ASSOC)['custom_class'])->toArray();


        foreach ($biblioCustomClassData??[] as $data) {
            foreach ($options as $button) {
                if (isset($button[0]) && $button[0] == $data)
                {
                    $buttonList .= '<input type="hidden" id="customClassFor'.$button[0].'" name="custom_class[]" value="' . $button[0] . '"/>';
                    $buttonList .= '<button class="btn btn-outline-secondary btn-sm rounded-pill m-1 px-2 py-1 deleteBtn" data-id="' . $button[0] . '">' . $button[1] . '<i class="ml-1 fa fa-close"></i></button>';
                }
            }
        }
    }

    // Kelompok Kustom
    $form->addAnything('Pengklompokan Kustom', <<<HTML
    <div class="w-100 d-flex flex-column">
        <div class="w-100">
            <select class="select select2 kelompok">
                {$list}
            </select>
        </div>
        <div id="resultArea" class="w-100 d-flex flex-wrap p-3 border border-secondary">
            {$buttonList}
        </div>
    </div>
    HTML);

    $js = <<<JS
    $('.kelompok').change(function(){
        // return;
        let target = $(this)[0].selectedOptions[0]
        let valueSelected = target.value
        let labelSelected = target.innerText
        
        if ($('#customClassFor' + valueSelected).length == 0)
        {
            $('#resultArea').append(`
                <input type="hidden" id="customClassFor\${valueSelected}" name="custom_class[]" value="\${valueSelected}"/>
                <button class="btn btn-outline-secondary btn-sm rounded-pill m-1 px-2 py-1 deleteBtn" data-id="\${valueSelected}">\${labelSelected}<i class="ml-1 fa fa-close"></i></button>
            `)
        }
    })

    $('#mainContent').on('click', '.deleteBtn', function(){
        let id = $(this).data('id')
        $('#customClassFor' + id).remove();
        $(this).remove();
    })
    JS;
});
```
##### Membuat direktori migration dan file 1_CreateTable.php
Buat direktori migration pada direktori book_custom_class/, setelah itu buat file 1_CreateTable.php pada direktori book_custom_class/migration/
```php
<?php
/**
 * @author Drajat Hasan
 * @email drajathasan20@gmail.com
 * @create date 2022-11-28 14:55:28
 * @modify date 2022-11-28 15:23:38
 * @license GPLv3
 * @desc [description]
 */

use SLiMS\Migration\Migration;
use SLiMS\Table\{Schema,Blueprint};

class CreateTable extends Migration
{
    function up()
    {
        Schema::create('mst_custom_class', function(Blueprint $table){
            $table->autoIncrement('id');
            $table->string('name', 50);
            $table->timestamps();
        });

        if (!Schema::hasColumn('biblio_custom', 'custom_class'))
        {
            Schema::table('biblio_custom', function(Blueprint $table){
                $table->text('custom_class')->nullable()->add();
            });
        }
    }

    function down()
    {
        
    }
}
```
