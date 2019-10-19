You need to create new position.

File:

admin/language/en-gb/design/layout.php
Find:

$_['text_content_bottom']
Add before it:

$_['text_content_new']    = 'Content New';
File:

admin/view/template/design/layout_form.twig
Find:

<table id="module-content-top" class="table table-striped table-bordered table-hover">
Add before it:

<table id="module-content-new" class="table table-striped table-bordered table-hover">
  <thead>
    <tr>
      <td class="text-center">{{ text_content_new }}</td>
    </tr>
  </thead>
  <tbody>
    {% for layout_module in layout_modules %}
    {% if layout_module.position == 'content_new' %}
    <tr id="module-row{{ module_row }}">
      <td class="text-left"><div class="input-group">
          <select name="layout_module[{{ module_row }}][code]" class="form-control input-sm">
            {% for extension in extensions %}
            <optgroup label="{{ extension.name }}">
            {% if not extension.module %}
            {% if extension.code == layout_module.code %}
            <option value="{{ extension.code }}" selected="selected">{{ extension.name }}</option>
            {% else %}
            <option value="{{ extension.code }}">{{ extension.name }}</option>
            {% endif %}
            {% else %}
            {% for module in extension.module %}
            {% if module.code == layout_module.code %}
            <option value="{{ module.code }}" selected="selected">{{ module.name }}</option>
            {% else %}
            <option value="{{ module.code }}">{{ module.name }}</option>
            {% endif %}
            {% endfor %}
            {% endif %}
            </optgroup>
            {% endfor %}
          </select>
          <input type="hidden" name="layout_module[{{ module_row }}][position]" value="{{ layout_module.position }}" />
          <input type="hidden" name="layout_module[{{ module_row }}][sort_order]" value="{{ layout_module.sort_order }}" />
          <div class="input-group-btn"> <a href="{{ layout_module.edit }}" type="button" data-toggle="tooltip" title="{{ button_edit }}" target="_blank" class="btn btn-primary btn-sm"><i class="fa fa-pencil"></i></a>
            <button type="button" onclick="$('#module-row{{ module_row }}').remove();" data-toggle="tooltip" title="{{ button_remove }}" class="btn btn-danger btn-sm"><i class="fa fa fa-minus-circle"></i></button>
          </div>
        </div></td>
    </tr>
    {% set module_row = module_row + 1 %}
    {% endif %}
    {% endfor %}
  </tbody>
  <tfoot>
    <tr>
      <td class="text-left"><div class="input-group">
          <select class="form-control input-sm">
            <option value=""></option>
            {% for extension in extensions %}
            <optgroup label="{{ extension.name }}">
            {% if not extension.module %}
            <option value="{{ extension.code }}">{{ extension.name }}</option>
            {% else %}
            {% for module in extension.module %}
            <option value="{{ module.code }}">{{ module.name }}</option>
            {% endfor %}
            {% endif %}
            </optgroup>
            {% endfor %}
          </select>
          <div class="input-group-btn">
            <button type="button" onclick="addModule('content-new');" data-toggle="tooltip" title="{{ button_module_add }}" class="btn btn-primary btn-sm"><i class="fa fa-plus-circle"></i></button>
          </div>
        </div></td>
    </tr>
  </tfoot>
</table>
Create the following file:

catalog/controller/common/content_new.php
it's content:

<?php
class ControllerCommonContentNew extends Controller {
    public function index() {
        $this->load->model('design/layout');
        if (isset($this->request->get['route'])) {
            $route = (string)$this->request->get['route'];
        } else {
            $route = 'common/home';
        }
        $layout_id = 0;
        if ($route == 'product/category' && isset($this->request->get['path'])) {
            $this->load->model('catalog/category');
            $path = explode('_', (string)$this->request->get['path']);
            $layout_id = $this->model_catalog_category->getCategoryLayoutId(end($path));
        }
        if ($route == 'product/product' && isset($this->request->get['product_id'])) {
            $this->load->model('catalog/product');
            $layout_id = $this->model_catalog_product->getProductLayoutId($this->request->get['product_id']);
        }
        if ($route == 'information/information' && isset($this->request->get['information_id'])) {
            $this->load->model('catalog/information');
            $layout_id = $this->model_catalog_information->getInformationLayoutId($this->request->get['information_id']);
        }
        if (!$layout_id) {
            $layout_id = $this->model_design_layout->getLayout($route);
        }
        if (!$layout_id) {
            $layout_id = $this->config->get('config_layout_id');
        }
        $this->load->model('setting/module');
        $data['modules'] = array();
        $modules = $this->model_design_layout->getLayoutModules($layout_id, 'content_new');
        foreach ($modules as $module) {
            $part = explode('.', $module['code']);
            if (isset($part[0]) && $this->config->get('module_' . $part[0] . '_status')) {
                $module_data = $this->load->controller('extension/module/' . $part[0]);
                if ($module_data) {
                    $data['modules'][] = $module_data;
                }
            }
            if (isset($part[1])) {
                $setting_info = $this->model_setting_module->getModule($part[1]);
                if ($setting_info && $setting_info['status']) {
                    $output = $this->load->controller('extension/module/' . $part[0], $setting_info);
                    if ($output) {
                        $data['modules'][] = $output;
                    }
                }
            }
        }
        return $this->load->view('common/content_new', $data);
    }
}
Create the following file:

catalog/view/theme/default/template/common/content_new.twig
it's content:

{% for module in modules %}
{{ module }}
{% endfor %}
File:

catalog/controller/common/home.php
Find:

$data['content_top'] = $this->load->controller('common/content_top');
Add before it:

$data['content_new'] = $this->load->controller('common/content_new');
File:

catalog/view/theme/default/template/common/home.twig
Find:

{{ header }}
Add after it:

{{ content_new }}